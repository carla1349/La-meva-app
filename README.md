# La-meva-app                
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plan Perfecto</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        :root {
            --color-primary: #8d6e63;
            --color-secondary: #d7ccc8;
            --color-accent: #5d4037;
            --color-bg: #fdfaf6;
        }

        body {
            background-color: var(--color-bg);
            color: var(--color-accent);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            overflow-x: hidden;
        }

        .header-gradient {
            background: linear-gradient(to right, #a1887f, #8d6e63);
        }

        .card {
            background: white;
            border-radius: 15px;
            box-shadow: 0 4px 15px rgba(141, 110, 99, 0.15);
        }

        .btn-primary {
            background-color: var(--color-primary);
            color: white;
            transition: all 0.3s;
        }

        .btn-primary:hover {
            background-color: var(--color-accent);
            transform: translateY(-2px);
        }

        .star-rating {
            color: #ffb300;
        }

        /* Collage Styles */
        .collage-container {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
            gap: 15px;
            padding: 20px;
        }

        .collage-item {
            position: relative;
            overflow: hidden;
            border-radius: 10px;
            aspect-ratio: 1 / 1;
            transition: transform 0.3s;
            border: 4px solid white;
            box-shadow: 0 4px 10px rgba(0,0,0,0.1);
        }

        .collage-item:hover {
            transform: scale(1.05) rotate(2deg);
            z-index: 10;
        }

        .collage-item img {
            width: 100%;
            height: 100%;
            object-fit: cover;
        }

        /* Flower Decoration */
        .flower-icon {
            color: #d7ccc8;
            position: absolute;
            opacity: 0.6;
            pointer-events: none;
            animation: float 6s ease-in-out infinite;
        }

        @keyframes float {
            0%, 100% { transform: translateY(0) rotate(0deg); }
            50% { transform: translateY(-20px) rotate(20deg); }
        }

        #historyModal {
            display: none;
            position: fixed;
            inset: 0;
            background: rgba(0,0,0,0.5);
            z-index: 50;
            align-items: center;
            justify-content: center;
        }

        select {
            cursor: pointer;
        }
    </style>
</head>
<body class="min-h-screen relative">

    <!-- Flores decorativas de fondo -->
    <i class="fas fa-seedling flower-icon text-4xl" style="top: 15%; left: 5%; animation-delay: 0s;"></i>
    <i class="fas fa-spa flower-icon text-3xl" style="top: 40%; right: 5%; animation-delay: 2s;"></i>
    <i class="fas fa-leaf flower-icon text-2xl" style="bottom: 20%; left: 8%; animation-delay: 4s;"></i>

    <!-- CABECERA -->
    <header class="header-gradient text-white p-6 shadow-lg relative z-10">
        <div class="max-w-6xl mx-auto flex justify-between items-center">
            <button onclick="toggleHistory()" class="flex items-center space-x-2 hover:opacity-80 transition">
                <div class="bg-white p-2 rounded-full">
                    <i class="fas fa-book-open text-amber-800 text-xl"></i>
                </div>
                <span id="txt-history-btn" class="font-semibold hidden sm:inline">Historial</span>
            </button>

            <div class="text-center">
                <h1 class="text-3xl md:text-4xl font-serif font-bold uppercase tracking-widest">Plan Perfecto</h1>
                <p id="txt-motto" class="italic text-sm md:text-base mt-2 opacity-90">"Cada día es una nueva oportunidad para crear recuerdos inolvidables"</p>
            </div>

            <div class="flex items-center space-x-2">
                <i class="fas fa-globe"></i>
                <select id="lang-selector" onchange="changeLanguage()" class="bg-transparent border border-white/40 rounded px-2 py-1 text-sm outline-none">
                    <option value="es" class="text-black">Castellano</option>
                    <option value="ca" class="text-black">Català</option>
                    <option value="en" class="text-black">English</option>
                </select>
            </div>
        </div>
    </header>

    <!-- CUERPO PRINCIPAL -->
    <main class="max-w-4xl mx-auto p-6 mt-8 relative z-10">
        <div class="card p-8 space-y-8">
            <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
                <div class="flex flex-col space-y-2">
                    <label class="font-bold flex items-center space-x-2">
                        <i class="far fa-calendar-alt text-amber-700"></i>
                        <span id="lbl-date">Fecha</span>
                    </label>
                    <input type="date" id="input-date" class="border border-secondary rounded-lg p-3 outline-none focus:ring-2 focus:ring-primary">
                </div>

                <div class="flex flex-col space-y-2">
                    <label class="font-bold flex items-center space-x-2">
                        <i class="fas fa-thermometer-half text-amber-700"></i>
                        <span id="lbl-temp">Temperatura (°C)</span>
                    </label>
                    <input type="number" id="input-temp" placeholder="Ej: 22" class="border border-secondary rounded-lg p-3 outline-none focus:ring-2 focus:ring-primary">
                </div>

                <div class="flex flex-col space-y-2">
                    <label class="font-bold flex items-center space-x-2">
                        <i class="fas fa-users text-amber-700"></i>
                        <span id="lbl-with">¿Con quién vas?</span>
                    </label>
                    <select id="input-with" class="border border-secondary rounded-lg p-3 outline-none focus:ring-2 focus:ring-primary bg-white">
                    </select>
                </div>
            </div>

            <div class="text-center pt-4">
                <button onclick="generatePlan()" class="btn-primary px-10 py-4 rounded-full text-xl font-bold uppercase tracking-wider shadow-md">
                    <i class="fas fa-magic mr-2"></i> <span id="btn-generate">Encontrar Plan</span>
                </button>
            </div>
        </div>

        <!-- RESULTADO DEL PLAN -->
        <div id="plan-result" class="hidden mt-12 card p-8 border-2 border-primary text-center animate-bounce-short">
            <h3 class="text-2xl font-bold mb-4 text-amber-900" id="txt-suggested-title">Tu plan ideal es...</h3>
            <div id="suggested-activity" class="text-3xl font-serif text-amber-800 mb-8 italic"></div>
            
            <div class="flex flex-wrap justify-center gap-4">
                <button onclick="acceptPlan()" class="bg-green-600 hover:bg-green-700 text-white px-8 py-3 rounded-lg font-bold transition shadow-sm">
                    <i class="fas fa-check mr-2"></i> <span id="btn-do-it">¡Lo hago!</span>
                </button>
                <button onclick="generatePlan()" class="bg-amber-600 hover:bg-amber-700 text-white px-8 py-3 rounded-lg font-bold transition shadow-sm">
                    <i class="fas fa-redo mr-2"></i> <span id="btn-other">Otro plan</span>
                </button>
            </div>
        </div>

        <!-- COLLAGE DE FOTOS Y FLORES -->
        <section class="mt-20">
            <div class="flex items-center justify-center space-x-4 mb-6">
                <div class="h-px bg-secondary flex-1"></div>
                <h2 class="text-2xl font-serif italic text-amber-800 flex items-center">
                    <i class="fas fa-spa mr-2 text-primary"></i> <span id="txt-inspire">Inspírate</span> <i class="fas fa-spa ml-2 text-primary"></i>
                </h2>
                <div class="h-px bg-secondary flex-1"></div>
            </div>
            
            <div class="collage-container">
                <!-- Imagen 1: Naturaleza -->
                <div class="collage-item">
                    <img src="https://images.unsplash.com/photo-1501785888041-af3ef285b470?auto=format&fit=crop&w=300&q=80" alt="Naturaleza">
                </div>
                <!-- Imagen 2: Cine (Actualizada) -->
                <div class="collage-item">
                    <img src="https://images.unsplash.com/photo-1485846234645-a62644f84728?auto=format&fit=crop&w=300&q=80" alt="Cine">
                </div>
                <!-- Imagen 3: Picnic -->
                <div class="collage-item">
                    <img src="https://images.unsplash.com/photo-1523301343968-6a6ebf63c672?auto=format&fit=crop&w=300&q=80" alt="Picnic">
                </div>
                <!-- Imagen 4: Café (Actualizada) -->
                <div class="collage-item">
                    <img src="https://images.unsplash.com/photo-1495474472287-4d71bcdd2085?auto=format&fit=crop&w=300&q=80" alt="Tomar Café">
                </div>
                <div class="collage-item hidden md:block">
                    <img src="https://images.unsplash.com/photo-1526047932273-341f2a7631f9?auto=format&fit=crop&w=300&q=80" alt="Flores">
                </div>
                <div class="collage-item hidden md:block">
                    <img src="https://images.unsplash.com/photo-1516627145497-ae6968895b74?auto=format&fit=crop&w=300&q=80" alt="Helado">
                </div>
            </div>
            
            <div class="text-center mt-6">
                <i class="fas fa-asterisk text-secondary text-xs mx-1"></i>
                <i class="fas fa-asterisk text-secondary text-xs mx-1"></i>
                <i class="fas fa-asterisk text-secondary text-xs mx-1"></i>
            </div>
        </section>
    </main>

    <footer class="text-center p-8 text-amber-800 opacity-60 text-sm">
        <p>Plan Perfecto &copy; 2024 - Hecho con amor y flores</p>
    </footer>

    <!-- MODAL DE HISTORIAL -->
    <div id="historyModal">
        <div class="bg-white rounded-2xl w-full max-w-2xl max-h-[80vh] overflow-hidden flex flex-col m-4 shadow-2xl">
            <div class="p-6 header-gradient text-white flex justify-between items-center">
                <h2 class="text-2xl font-bold flex items-center">
                    <i class="fas fa-history mr-3"></i> <span id="lbl-history-title">Mis Planes Guardados</span>
                </h2>
                <button onclick="toggleHistory()" class="text-3xl">&times;</button>
            </div>
            
            <div id="history-content" class="p-6 overflow-y-auto flex-1 space-y-4">
            </div>

            <div class="p-4 bg-gray-50 border-t flex justify-between">
                <button onclick="clearHistory()" class="text-red-600 hover:text-red-800 font-bold text-sm uppercase">
                    <i class="fas fa-trash-alt mr-1"></i> <span id="btn-clear-all">Borrar Historial</span>
                </button>
                <button onclick="toggleHistory()" class="btn-primary px-6 py-2 rounded font-bold text-sm uppercase">
                    <span id="btn-close">Cerrar</span>
                </button>
            </div>
        </div>
    </div>

    <script>
        let currentPlan = null;
        let history = JSON.parse(localStorage.getItem('planHistory')) || [];

        const translations = {
            es: {
                motto: '"Cada día es una nueva oportunidad para crear recuerdos inolvidables"',
                historyBtn: "Historial",
                date: "Fecha",
                temp: "Temperatura (°C)",
                with: "¿Con quién vas?",
                generate: "Encontrar Plan",
                suggested: "Tu plan ideal es...",
                doIt: "¡Lo hago!",
                other: "Otro plan",
                historyTitle: "Mis Planes Guardados",
                clearAll: "Borrar Historial",
                close: "Cerrar",
                noPlans: "Aún no has guardado ningún plan.",
                optionsWith: ["Pareja", "Amigos", "Solo", "Familia"],
                inspire: "Inspírate",
                plans: [
                    "Ir a tomar un helado artesano",
                    "Paseo por la montaña",
                    "Tarde de películas y palomitas en casa",
                    "Visitar un museo local",
                    "Cena romántica en un restaurante nuevo",
                    "Ir a la playa a ver el atardecer",
                    "Hacer un picnic en el parque",
                    "Sesión de juegos de mesa",
                    "Ir a una cafetería a leer un libro",
                    "Hacer una ruta en bicicleta",
                    "Ir a patinar",
                    "Cocinar una receta nueva y elaborada"
                ]
            },
            ca: {
                motto: '"Cada dia és una nova oportunitat per crear records inoblidables"',
                historyBtn: "Historial",
                date: "Data",
                temp: "Temperatura (°C)",
                with: "Amb qui vas?",
                generate: "Trobar Pla",
                suggested: "El teu pla ideal és...",
                doIt: "Ho faig!",
                other: "Un altre pla",
                historyTitle: "Els Meus Plans Guardats",
                clearAll: "Esborrar Historial",
                close: "Tancar",
                noPlans: "Encara no has guardat cap pla.",
                optionsWith: ["Parella", "Amics", "Sol", "Família"],
                inspire: "Inspira't",
                plans: [
                    "Anar a prendre un gelat artesà",
                    "Passejada per la muntanya",
                    "Tarda de pel·lícules i crispetes a casa",
                    "Visitar un museu local",
                    "Sopar romàntic en un restaurant nou",
                    "Anar a la platja a veure la posta de sol",
                    "Fer un pícnic al parc",
                    "Sessió de jocs de taula",
                    "Anar a una cafeteria a llegir un llibre",
                    "Fer una ruta en bicicleta",
                    "Anar a patinar",
                    "Cuinar una recepta nova i elaborada"
                ]
            },
            en: {
                motto: '"Every day is a new opportunity to create unforgettable memories"',
                historyBtn: "History",
                date: "Date",
                temp: "Temperature (°C)",
                with: "Who are you with?",
                generate: "Find a Plan",
                suggested: "Your ideal plan is...",
                doIt: "I'll do it!",
                other: "Other plan",
                historyTitle: "My Saved Plans",
                clearAll: "Clear History",
                close: "Close",
                noPlans: "You haven't saved any plans yet.",
                optionsWith: ["Partner", "Friends", "Alone", "Family"],
                inspire: "Get Inspired",
                plans: [
                    "Go get an artisan ice cream",
                    "A walk in the mountains",
                    "Movie and popcorn afternoon at home",
                    "Visit a local museum",
                    "Romantic dinner at a new restaurant",
                    "Go to the beach to watch the sunset",
                    "Have a picnic in the park",
                    "Board game session",
                    "Go to a cafe to read a book",
                    "Go on a bike route",
                    "Go skating",
                    "Cook a new and elaborate recipe"
                ]
            }
        };

        function changeLanguage() {
            const lang = document.getElementById('lang-selector').value;
            const t = translations[lang];

            document.getElementById('txt-motto').innerText = t.motto;
            document.getElementById('txt-history-btn').innerText = t.historyBtn;
            document.getElementById('lbl-date').innerText = t.date;
            document.getElementById('lbl-temp').innerText = t.temp;
            document.getElementById('lbl-with').innerText = t.with;
            document.getElementById('btn-generate').innerText = t.generate;
            document.getElementById('txt-suggested-title').innerText = t.suggested;
            document.getElementById('btn-do-it').innerText = t.doIt;
            document.getElementById('btn-other').innerText = t.other;
            document.getElementById('lbl-history-title').innerText = t.historyTitle;
            document.getElementById('btn-clear-all').innerText = t.clearAll;
            document.getElementById('btn-close').innerText = t.close;
            document.getElementById('txt-inspire').innerText = t.inspire;

            const withSelector = document.getElementById('input-with');
            withSelector.innerHTML = '';
            t.optionsWith.forEach(opt => {
                const option = document.createElement('option');
                option.value = opt;
                option.innerText = opt;
                withSelector.appendChild(option);
            });
        }

        function generatePlan() {
            const lang = document.getElementById('lang-selector').value;
            const plans = translations[lang].plans;
            const randomPlan = plans[Math.floor(Math.random() * plans.length)];
            
            currentPlan = randomPlan;
            document.getElementById('suggested-activity').innerText = randomPlan;
            document.getElementById('plan-result').classList.remove('hidden');
            
            document.getElementById('plan-result').scrollIntoView({ behavior: 'smooth' });
        }

        function acceptPlan() {
            const date = document.getElementById('input-date').value || new Date().toISOString().split('T')[0];
            const temp = document.getElementById('input-temp').value || "--";
            const withWho = document.getElementById('input-with').value;
            
            const newRecord = {
                id: Date.now(),
                plan: currentPlan,
                date: date,
                temp: temp,
                withWho: withWho,
                rating: 0
            };

            history.unshift(newRecord);
            saveHistory();
            
            const btn = document.getElementById('btn-do-it');
            btn.innerText = "¡Guardado!";
            setTimeout(() => {
                btn.innerText = translations[document.getElementById('lang-selector').value].doIt;
                document.getElementById('plan-result').classList.add('hidden');
                toggleHistory();
            }, 800);
        }

        function toggleHistory() {
            const modal = document.getElementById('historyModal');
            if (modal.style.display === 'flex') {
                modal.style.display = 'none';
            } else {
                renderHistory();
                modal.style.display = 'flex';
            }
        }

        function renderHistory() {
            const container = document.getElementById('history-content');
            const lang = document.getElementById('lang-selector').value;
            container.innerHTML = '';

            if (history.length === 0) {
                container.innerHTML = `<p class="text-center text-gray-500 italic py-10">${translations[lang].noPlans}</p>`;
                return;
            }

            history.forEach(item => {
                const card = document.createElement('div');
                card.className = "bg-orange-50 p-4 rounded-xl border border-orange-100 relative";
                
                let stars = '';
                for(let i=1; i<=5; i++) {
                    stars += `<i class="${i <= item.rating ? 'fas' : 'far'} fa-star star-rating cursor-pointer" onclick="ratePlan(${item.id}, ${i})"></i>`;
                }

                card.innerHTML = `
                    <button onclick="deletePlan(${item.id})" class="absolute top-2 right-2 text-gray-400 hover:text-red-500 transition">
                        <i class="fas fa-times-circle"></i>
                    </button>
                    <h4 class="font-bold text-amber-900 text-lg">${item.plan}</h4>
                    <div class="grid grid-cols-3 gap-2 mt-2 text-xs text-amber-800 opacity-80">
                        <span><i class="far fa-calendar-alt mr-1"></i> ${item.date}</span>
                        <span><i class="fas fa-thermometer-half mr-1"></i> ${item.temp}°C</span>
                        <span><i class="fas fa-users mr-1"></i> ${item.withWho}</span>
                    </div>
                    <div class="mt-3 flex items-center justify-between">
                        <div class="space-x-1">${stars}</div>
                    </div>
                `;
                container.appendChild(card);
            });
        }

        function ratePlan(id, rating) {
            history = history.map(item => item.id === id ? {...item, rating: rating} : item);
            saveHistory();
            renderHistory();
        }

        function deletePlan(id) {
            history = history.filter(item => item.id !== id);
            saveHistory();
            renderHistory();
        }

        function clearHistory() {
            if(confirm("¿Seguro que quieres borrar todo tu historial de planes?")) {
                history = [];
                saveHistory();
                renderHistory();
            }
        }

        function saveHistory() {
            localStorage.setItem('planHistory', JSON.stringify(history));
        }

        window.onload = () => {
            changeLanguage();
            document.getElementById('input-date').valueAsDate = new Date();
        };
    </script>
</body>
</html>
