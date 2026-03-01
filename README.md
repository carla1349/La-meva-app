# La-meva-app                
<!DOCTYPE html>
<html lang="ca">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pla Perfecte - Generador de moments</title>
    
    <!-- Estils Tailwind CSS via CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Fonts de Google -->
    <link href="https://fonts.googleapis.com/css2?family=Quicksand:wght@300;500;700&family=Playfair+Display:ital,wght@0,400;0,700;1,400&display=swap" rel="stylesheet">
    
    <!-- Icones Lucide -->
    <script src="https://unpkg.com/lucide@latest"></script>
    
    <!-- Firebase SDK -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, onAuthStateChanged, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, setDoc, collection, onSnapshot, updateDoc, deleteDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'pla-perfecte-v3';

        let currentUser = null;
        window.historyData = []; 

        const initAuth = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (error) {
                console.error("Error d'autenticació:", error);
            }
        };

        onAuthStateChanged(auth, (user) => {
            currentUser = user;
            if (user) {
                // RULE 1: Ruta específica per usuari per a historial privat
                const userPlansRef = collection(db, 'artifacts', appId, 'users', user.uid, 'plans');
                onSnapshot(userPlansRef, (snapshot) => {
                    window.historyData = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                    window.historyData.sort((a, b) => b.createdAt - a.createdAt);
                    window.updateHistoryUI();
                }, (error) => {
                    console.error("Error carregant plans:", error);
                });
            }
        });

        initAuth();

        window.savePlanToCloud = async (planEntry) => {
            if (!currentUser) return;
            const docId = planEntry.id.toString();
            const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'plans', docId);
            await setDoc(docRef, { ...planEntry, createdAt: Date.now() });
        };

        window.updatePlanRating = async (planId, rating) => {
            if (!currentUser) return;
            const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'plans', planId.toString());
            await updateDoc(docRef, { rating: rating });
        };

        window.deletePlan = async (planId) => {
            if (!currentUser) return;
            const docRef = doc(db, 'artifacts', appId, 'users', currentUser.uid, 'plans', planId.toString());
            await deleteDoc(docRef);
        };
    </script>

    <style>
        :root {
            --color-terra: #8d6e63;
            --color-crema: #fdfcfb;
            --color-sorra: #d7ccc8;
            --color-marro-fosc: #4e342e;
        }
        
        body {
            font-family: 'Quicksand', sans-serif;
            background-color: var(--color-crema);
            color: var(--color-marro-fosc);
            margin: 0;
            padding: 0;
        }

        .font-display { font-family: 'Playfair Display', serif; }

        .card {
            background: white;
            border: 1px solid var(--color-sorra);
            box-shadow: 0 10px 30px -10px rgba(141, 110, 99, 0.15);
            border-radius: 2rem;
        }

        .btn-primary {
            background-color: var(--color-terra);
            color: white;
            transition: all 0.3s ease;
        }

        .btn-primary:hover {
            background-color: var(--color-marro-fosc);
            transform: translateY(-2px);
        }

        .fade-in {
            animation: fadeIn 0.6s ease-out forwards;
        }

        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(10px); }
            to { opacity: 1; transform: translateY(0); }
        }

        .sidebar {
            transition: transform 0.4s cubic-bezier(0.4, 0, 0.2, 1);
            background: white;
            z-index: 100;
        }

        .sidebar-closed { transform: translateX(-100%); }
        .sidebar-open { transform: translateX(0); }

        .spinner {
            border: 2px solid rgba(141, 110, 99, 0.2);
            border-top: 2px solid var(--color-terra);
            border-radius: 50%;
            width: 16px;
            height: 16px;
            animation: spin 1s linear infinite;
        }

        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="min-h-screen flex flex-col">

    <!-- Sidebar Historial Individual -->
    <div id="sidebar" class="fixed top-0 left-0 h-full w-80 shadow-2xl sidebar-closed p-6 overflow-y-auto sidebar">
        <div class="flex justify-between items-center mb-8 border-b pb-4">
            <div>
                <h3 class="font-display text-xl font-bold">Els meus plans</h3>
                <p class="text-[10px] text-stone-400 uppercase tracking-widest mt-1">Historial personal</p>
            </div>
            <button onclick="toggleSidebar()" class="p-2 hover:bg-stone-100 rounded-full">
                <i data-lucide="x" class="w-5 h-5"></i>
            </button>
        </div>
        <div id="history-list" class="space-y-4">
            <!-- Cargado mediante Firestore -->
        </div>
    </div>

    <!-- Navegación -->
    <nav class="p-6 flex justify-between items-center max-w-6xl mx-auto w-full">
        <button onclick="toggleSidebar()" class="flex items-center gap-3 group">
            <div class="w-12 h-12 bg-white rounded-full flex items-center justify-center text-terra shadow-sm group-hover:shadow-md transition-all">
                <i data-lucide="bookmark" class="w-5 h-5" style="color:#8d6e63"></i>
            </div>
            <span class="text-[10px] font-bold uppercase tracking-widest text-stone-400 hidden sm:block">Historial</span>
        </button>

        <div class="flex items-center gap-4 bg-white/50 px-4 py-2 rounded-full border border-stone-200">
            <i data-lucide="languages" class="w-4 h-4 text-stone-400"></i>
            <select id="lang" onchange="updateUI()" class="bg-transparent text-sm font-medium outline-none cursor-pointer">
                <option value="ca">Català</option>
                <option value="es">Castellano</option>
                <option value="en">English</option>
            </select>
        </div>
    </nav>

    <!-- Contenido -->
    <main class="flex-grow flex flex-col items-center justify-center px-6 py-12">
        <header class="text-center mb-12 max-w-2xl fade-in">
            <h1 id="main-title" class="font-display text-5xl md:text-7xl font-bold mb-4">Pla Perfecte</h1>
            <!-- Frase debajo del título -->
            <p id="quote" class="italic text-stone-500 text-lg md:text-xl font-light leading-relaxed">Carregant inspiració...</p>
        </header>

        <div class="grid lg:grid-cols-2 gap-8 w-full max-w-5xl items-stretch">
            
            <!-- Configuración -->
            <section class="card p-8 md:p-10 space-y-6 flex flex-col justify-between">
                <div>
                    <h2 id="label-config" class="text-2xl font-bold text-stone-800 mb-6">Configura el teu dia</h2>
                    <div class="space-y-4">
                        <div>
                            <label id="label-date" class="block text-xs font-bold text-stone-400 uppercase tracking-widest mb-2">Quan vols anar-hi?</label>
                            <input type="date" id="dateInput" class="w-full p-4 rounded-xl border border-stone-200 outline-none">
                        </div>
                        <div>
                            <label id="label-temp" class="block text-xs font-bold text-stone-400 uppercase tracking-widest mb-2">Quin temps fa?</label>
                            <select id="tempInput" class="w-full p-4 rounded-xl border border-stone-200 outline-none"></select>
                        </div>
                        <div>
                            <label id="label-who" class="block text-xs font-bold text-stone-400 uppercase tracking-widest mb-2">Amb qui vas?</label>
                            <select id="whoInput" class="w-full p-4 rounded-xl border border-stone-200 outline-none"></select>
                        </div>
                    </div>
                </div>
                <button id="btn-generate" onclick="generatePlan()" class="btn-primary w-full py-5 mt-6 rounded-2xl font-bold uppercase tracking-widest text-sm shadow-lg">
                    Generar Pla
                </button>
            </section>

            <!-- Resultados -->
            <section id="result-area" class="hidden h-full">
                <div class="card p-8 md:p-10 flex flex-col items-center justify-center text-center h-full fade-in">
                    <h3 id="label-result-title" class="font-display text-3xl font-bold text-terra mb-6">El Teu Pla</h3>
                    <p id="plan-text" class="text-2xl text-stone-700 italic font-medium leading-relaxed mb-10"></p>
                    
                    <div id="ai-box" class="hidden w-full bg-stone-50 p-6 rounded-2xl border border-stone-100 text-left mb-8">
                        <div class="flex items-center gap-2 mb-3 text-terra">
                            <i data-lucide="sparkles" class="w-4 h-4"></i>
                            <span class="text-[10px] font-bold uppercase tracking-tighter">Detalls amb IA</span>
                        </div>
                        <p id="ai-text" class="text-sm text-stone-600 leading-relaxed"></p>
                    </div>

                    <div class="flex flex-wrap justify-center gap-4">
                        <!-- Opción "Lo haré" -->
                        <button id="btn-save" onclick="saveCurrentPlan()" class="flex items-center gap-2 bg-green-50 text-green-700 px-6 py-4 rounded-full border border-green-100 hover:bg-green-100 transition-all font-bold text-xs uppercase">
                            <i data-lucide="check" class="w-4 h-4"></i>
                            <span id="label-save">Ho faré!</span>
                        </button>
                        
                        <!-- Opción "Otro plan" -->
                        <button onclick="generatePlan()" class="flex items-center gap-2 bg-stone-50 text-stone-700 px-6 py-4 rounded-full border border-stone-100 hover:bg-stone-100 transition-all font-bold text-xs uppercase">
                            <i data-lucide="refresh-cw" class="w-4 h-4"></i>
                            <span id="label-another">Un altre pla</span>
                        </button>

                        <!-- Botón IA opcional -->
                        <button id="btn-ai" onclick="personalizeWithAI()" class="flex items-center gap-2 bg-purple-50 text-purple-700 px-6 py-4 rounded-full border border-purple-100 hover:bg-purple-100 transition-all font-bold text-xs uppercase">
                            <i data-lucide="sparkles" class="w-4 h-4" id="ai-icon"></i>
                            <span id="label-ai">Més detalls</span>
                        </button>
                    </div>
                </div>
            </section>
        </div>
    </main>

    <script>
        const apiKey = ""; 
        let currentPlanText = "";
        let isAiThinking = false;

        const translations = {
            ca: {
                mainTitle: "Pla Perfecte",
                config: "Configura el teu dia",
                date: "Quan vols anar-hi?",
                temp: "Quin temps fa?",
                who: "Amb qui vas?",
                btnGen: "Generar Pla",
                resTitle: "El teu moment ideal",
                save: "Ho faré!",
                another: "Un altre pla",
                ai: "Més detalls ✨",
                saved: "Guardat!",
                temps: { cold: "Fa fred", mild: "Temperat", hot: "Fa calor" },
                whos: { solo: "Sol/a", friends: "Amics", couple: "En parella", family: "Família" },
                plans: {
                    cold_solo: ["Visitar una llibreria antiga i prendre un te.", "Anar al cine a veure una peli clàssica.", "Fer un trencaclosques en una cafeteria acollidora."],
                    mild_solo: ["Passejar pel barri vell i fer fotos.", "Llegir un llibre al parc sota un arbre.", "Pícnic solitari vora el mar."],
                    hot_solo: ["Bany matinal a la platja i gelat.", "Anar a un museu amb aire condicionat.", "Llegir a la fresca d'un pati interior."],
                    cold_friends: ["Tarda de jocs de taula i xocolata desfet.", "Anar a un Escape Room interior.", "Sopar en un racó amb llar de foc."],
                    mild_friends: ["Ruta de senderisme suau.", "Sopar de tapes en una terrassa.", "Fer una volta en bicicleta."],
                    hot_friends: ["Dia de platja i pícnic de fruita.", "Anar a un parc aquàtic.", "Sopar a la fresca vora la piscina."],
                    cold_couple: ["Marató de cinema a casa amb crispetes.", "Sopar romàntic en un bistro íntim.", "Passejada nocturna ben tapats."],
                    mild_couple: ["Passejada per un jardí botànic.", "Pícnic al capvespre amb vi blanc.", "Visitar un poble medieval."],
                    hot_couple: ["Bany nocturn a la platja sota els estels.", "Sopar a la fresca amb música en directe.", "Cine d'estiu a la fresca."],
                    cold_family: ["Tarda de cuina casolana (galetes!).", "Visitar un museu interactiu.", "Anar al teatre familiar."],
                    mild_family: ["Excursió a una granja escola.", "Jocs al parc i berenar.", "Visitar un castell."],
                    hot_family: ["Dia de piscina i gelats casolans.", "Excursió a un riu fresquet.", "Cine d'estiu familiar."]
                }
            },
            es: {
                mainTitle: "Plan Perfecto",
                config: "Configura tu día",
                date: "¿Cuándo quieres ir?",
                temp: "¿Qué tiempo hace?",
                who: "¿Con quién vas?",
                btnGen: "Generar Plan",
                resTitle: "Tu momento ideal",
                save: "¡Lo haré!",
                another: "Otro plan",
                ai: "Más detalles ✨",
                saved: "¡Guardado!",
                temps: { cold: "Hace frío", mild: "Templado", hot: "Hace calor" },
                whos: { solo: "Solo/a", friends: "Amigos", couple: "En pareja", family: "Familia" }
            },
            en: {
                mainTitle: "Perfect Plan",
                config: "Configure your day",
                date: "When are you going?",
                temp: "What's the weather?",
                who: "Who are you with?",
                btnGen: "Generate Plan",
                resTitle: "Your ideal moment",
                save: "I'll do it!",
                another: "Other plan",
                ai: "More details ✨",
                saved: "Saved!",
                temps: { cold: "Cold", mild: "Mild", hot: "Hot" },
                whos: { solo: "Solo", friends: "Friends", couple: "Couple", family: "Family" }
            }
        };

        async function apiCall(prompt, system) {
            let delay = 1000;
            for (let i = 0; i < 5; i++) {
                try {
                    const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                        method: 'POST',
                        headers: { 'Content-Type': 'application/json' },
                        body: JSON.stringify({
                            contents: [{ parts: [{ text: prompt }] }],
                            systemInstruction: { parts: [{ text: system }] }
                        })
                    });
                    const data = await res.json();
                    return data.candidates?.[0]?.content?.parts?.[0]?.text;
                } catch (e) {
                    await new Promise(r => setTimeout(r, delay));
                    delay *= 2;
                }
            }
            return null;
        }

        async function getInspiration() {
            const lang = document.getElementById('lang').value;
            const quote = await apiCall(`Genera una frase inspiradora molt curta (màxim 12 paraules) sobre viure el moment i gaudir del dia en idioma ${lang}. No facis servir cometes.`, "Ets un expert en motivació i benestar.");
            if (quote) document.getElementById('quote').innerText = quote.trim();
        }

        function updateUI() {
            const l = document.getElementById('lang').value;
            const t = translations[l];

            document.getElementById('main-title').innerText = t.mainTitle;
            document.getElementById('label-config').innerText = t.config;
            document.getElementById('label-date').innerText = t.date;
            document.getElementById('label-temp').innerText = t.temp;
            document.getElementById('label-who').innerText = t.who;
            document.getElementById('btn-generate').innerText = t.btnGen;
            document.getElementById('label-result-title').innerText = t.resTitle;
            document.getElementById('label-save').innerText = t.save;
            document.getElementById('label-another').innerText = t.another;
            document.getElementById('label-ai').innerText = t.ai;

            const tempSelect = document.getElementById('tempInput');
            const whoSelect = document.getElementById('whoInput');
            const oldT = tempSelect.value;
            const oldW = whoSelect.value;

            tempSelect.innerHTML = Object.entries(t.temps).map(([k,v]) => `<option value="${k}" ${k===oldT?'selected':''}>${v}</option>`).join('');
            whoSelect.innerHTML = Object.entries(t.whos).map(([k,v]) => `<option value="${k}" ${k===oldW?'selected':''}>${v}</option>`).join('');

            getInspiration();
        }

        function generatePlan() {
            const temp = document.getElementById('tempInput').value;
            const who = document.getElementById('whoInput').value;
            const key = `${temp}_${who}`;
            
            const availablePlans = translations.ca.plans[key];
            const randomPlan = availablePlans[Math.floor(Math.random() * availablePlans.length)];
            
            currentPlanText = randomPlan;
            document.getElementById('plan-text').innerText = randomPlan;
            document.getElementById('result-area').classList.remove('hidden');
            document.getElementById('ai-box').classList.add('hidden');
            
            lucide.createIcons();
        }

        async function personalizeWithAI() {
            if (isAiThinking) return;
            const l = document.getElementById('lang').value;
            const temp = document.getElementById('tempInput').selectedOptions[0].text;
            const who = document.getElementById('whoInput').selectedOptions[0].text;
            
            isAiThinking = true;
            document.getElementById('ai-icon').className = "spinner";
            
            const prompt = `Tinc aquest pla bàsic: "${currentPlanText}". El clima és ${temp} i estic amb ${who}. Dona'm 3 detalls màgics per elevar l'experiència. Respon en ${l}. Sigues poètic però breu.`;
            
            const result = await apiCall(prompt, "Ets un dissenyador d'experiències inoblidables.");
            
            if (result) {
                document.getElementById('ai-text').innerText = result;
                document.getElementById('ai-box').classList.remove('hidden');
            }
            
            isAiThinking = false;
            document.getElementById('ai-icon').className = "w-4 h-4";
            document.getElementById('ai-icon').setAttribute('data-lucide', 'sparkles');
            lucide.createIcons();
        }

        async function saveCurrentPlan() {
            const l = document.getElementById('lang').value;
            const planEntry = {
                id: Date.now(),
                text: currentPlanText,
                date: document.getElementById('dateInput').value,
                temp: document.getElementById('tempInput').selectedOptions[0].text,
                who: document.getElementById('whoInput').selectedOptions[0].text,
                rating: 5
            };

            await window.savePlanToCloud(planEntry);
            
            const btnLabel = document.getElementById('label-save');
            const original = btnLabel.innerText;
            btnLabel.innerText = translations[l].saved;
            setTimeout(() => btnLabel.innerText = original, 2000);
        }

        function toggleSidebar() {
            const s = document.getElementById('sidebar');
            s.classList.toggle('sidebar-closed');
            s.classList.toggle('sidebar-open');
        }

        window.updateHistoryUI = () => {
            const list = document.getElementById('history-list');
            if (!window.historyData || window.historyData.length === 0) {
                list.innerHTML = `<p class="text-stone-400 text-xs italic text-center mt-10">Encara no has guardat plans privats.</p>`;
                return;
            }

            list.innerHTML = window.historyData.map(item => `
                <div class="p-4 rounded-2xl border border-stone-100 bg-stone-50/50 relative group">
                    <button onclick="window.deletePlan('${item.id}')" class="absolute top-2 right-2 opacity-0 group-hover:opacity-100 transition-opacity text-stone-300 hover:text-red-400">
                        <i data-lucide="trash-2" class="w-3 h-3"></i>
                    </button>
                    <div class="flex justify-between text-[8px] font-bold text-stone-400 uppercase mb-1">
                        <span>${item.date}</span>
                        <span>${item.temp}</span>
                    </div>
                    <p class="text-sm text-stone-700 font-medium mb-2 pr-4">"${item.text}"</p>
                    <div class="flex gap-1">
                        ${[1,2,3,4,5].map(i => `
                            <button onclick="window.updatePlanRating('${item.id}', ${i})">
                                <i data-lucide="star" class="w-3 h-3 ${i <= (item.rating || 0) ? 'fill-yellow-400 text-yellow-400' : 'text-stone-200'}"></i>
                            </button>
                        `).join('')}
                    </div>
                </div>
            `).join('');
            lucide.createIcons();
        };

        window.onload = () => {
            document.getElementById('dateInput').valueAsDate = new Date();
            updateUI();
            lucide.createIcons();
        }
    </script>
</body>
</html>
