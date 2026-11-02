[index.html](https://github.com/user-attachments/files/34272939/index.html)
<!DOCTYPE html>
<html lang="it" class="h-full bg-slate-50">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MensaTrack - Gestione Pasti Scolastici & Report</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js per i grafici -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Font Awesome Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <!-- Google Fonts -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">

    <!-- html2pdf.js per esportazione diretta in PDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        brand: {
                            50: '#f0f9ff',
                            100: '#e0f2fe',
                            500: '#0284c7',
                            600: '#0369a1',
                            700: '#075985',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f5f9;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8;
        }
        @media print {
            body { background: white !important; }
            .no-print { display: none !important; }
            .print-only { display: block !important; }
            .print-card { border: none !important; shadow: none !important; }
        }
    </style>
</head>
<body class="h-full flex flex-col font-sans text-slate-800 antialiased bg-slate-50">

    <!-- Toast Notifications -->
    <div id="toast-container" class="fixed top-5 right-5 z-50 flex flex-col gap-2 pointer-events-none"></div>

    <!-- Navigation Bar -->
    <header class="bg-white border-b border-slate-200 sticky top-0 z-30 shadow-sm no-print">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between h-16 items-center">
                <div class="flex items-center gap-3">
                    <div class="bg-sky-600 text-white p-2.5 rounded-xl shadow-md shadow-sky-200">
                        <i class="fa-solid font-bold fa-school text-xl"></i>
                    </div>
                    <div>
                        <h1 class="text-xl font-bold text-slate-900 leading-tight">MensaTrack</h1>
                        <p class="text-xs text-slate-500 font-medium">Gestione Pasti Scolastici & Report</p>
                    </div>
                </div>
                
                <div class="flex items-center gap-2">
                    <button onclick="loadSampleData()" title="Carica Dati di Esempio" class="hidden sm:inline-flex items-center gap-2 px-3 py-2 text-xs font-semibold text-sky-700 bg-sky-50 hover:bg-sky-100 rounded-lg transition-colors border border-sky-200">
                        <i class="fa-solid fa-vial"></i>
                        <span>Dati Esempio</span>
                    </button>
                    <button onclick="exportDataCSV()" title="Esporta CSV" class="inline-flex items-center gap-2 px-3 py-2 text-xs font-semibold text-slate-700 bg-white hover:bg-slate-100 border border-slate-300 rounded-lg transition-colors shadow-sm">
                        <i class="fa-solid fa-file-csv text-emerald-600"></i>
                        <span class="hidden md:inline">Esporta CSV</span>
                    </button>
                    <button onclick="triggerImport()" title="Importa JSON" class="inline-flex items-center gap-2 px-3 py-2 text-xs font-semibold text-slate-700 bg-white hover:bg-slate-100 border border-slate-300 rounded-lg transition-colors shadow-sm">
                        <i class="fa-solid fa-file-import text-amber-600"></i>
                        <span class="hidden md:inline">Importa</span>
                    </button>
                    <input type="file" id="importFileInput" accept=".json" class="hidden" onchange="importDataJSON(event)">
                </div>
            </div>
        </div>
    </header>

    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 lg:px-8 py-8 space-y-8">

        <!-- Top Statistics KPI Cards -->
        <section class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 no-print">
            <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Totale Pasti Registrati</p>
                    <h3 id="stat-total" class="text-2xl font-bold text-slate-900 mt-1">0</h3>
                    <p class="text-xs text-slate-500 mt-1"><span id="stat-records-count">0</span> inserimenti</p>
                </div>
                <div class="w-12 h-12 bg-sky-50 text-sky-600 rounded-xl flex items-center justify-center text-xl font-bold border border-sky-100">
                    <i class="fa-solid fa-utensils"></i>
                </div>
            </div>

            <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Scuola Elementare</p>
                    <h3 id="stat-elementari" class="text-2xl font-bold text-indigo-600 mt-1">0</h3>
                    <p class="text-xs text-slate-500 mt-1">pasti erogati</p>
                </div>
                <div class="w-12 h-12 bg-indigo-50 text-indigo-600 rounded-xl flex items-center justify-center text-xl font-bold border border-indigo-100">
                    <i class="fa-solid fa-child"></i>
                </div>
            </div>

            <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Scuola Media</p>
                    <h3 id="stat-medie" class="text-2xl font-bold text-amber-600 mt-1">0</h3>
                    <p class="text-xs text-slate-500 mt-1">pasti erogati</p>
                </div>
                <div class="w-12 h-12 bg-amber-50 text-amber-600 rounded-xl flex items-center justify-center text-xl font-bold border border-amber-100">
                    <i class="fa-solid fa-graduation-cap"></i>
                </div>
            </div>

            <div class="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between">
                <div>
                    <p class="text-xs font-semibold text-slate-400 uppercase tracking-wider">Totale Dipendenti</p>
                    <h3 id="stat-dipendenti" class="text-2xl font-bold text-emerald-600 mt-1">0</h3>
                    <p class="text-xs text-slate-500 mt-1">pasti docenti/personale</p>
                </div>
                <div class="w-12 h-12 bg-emerald-50 text-emerald-600 rounded-xl flex items-center justify-center text-xl font-bold border border-emerald-100">
                    <i class="fa-solid fa-user-tie"></i>
                </div>
            </div>
        </section>

        <!-- Main Workspace: Insertion Form & Report Generator -->
        <div class="grid grid-cols-1 lg:grid-cols-12 gap-8 items-start">
            
            <!-- Left Column: Registration Form -->
            <div class="lg:col-span-5 bg-white rounded-2xl border border-slate-200 shadow-sm p-6 space-y-6 no-print">
                <div class="flex items-center justify-between border-b border-slate-100 pb-4">
                    <div class="flex items-center gap-2">
                        <i class="fa-solid fa-pen-to-square text-sky-600"></i>
                        <h2 id="form-title" class="text-lg font-bold text-slate-800">Inserimento Pasti</h2>
                    </div>
                    <button id="cancel-edit-btn" onclick="resetForm()" class="hidden text-xs font-semibold text-slate-400 hover:text-slate-600">
                        <i class="fa-solid fa-xmark"></i> Annulla
                    </button>
                </div>

                <form id="meal-form" onsubmit="handleFormSubmit(event)" class="space-y-4">
                    <input type="hidden" id="edit-id" value="">

                    <!-- Data Campo -->
                    <div>
                        <label for="input-date" class="block text-xs font-semibold text-slate-700 uppercase tracking-wider mb-1">Data *</label>
                        <input type="date" id="input-date" required class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-slate-800 text-sm focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white transition-all">
                    </div>

                    <!-- Scuola Campo (Pulsanti rapidi + Select) -->
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 uppercase tracking-wider mb-1">Scuola *</label>
                        <div class="grid grid-cols-3 gap-2 mb-2">
                            <button type="button" onclick="selectSchool('Elementare')" class="school-btn text-xs py-2 px-2 rounded-lg border border-slate-200 font-medium text-slate-600 hover:bg-slate-100 transition flex flex-col items-center gap-1" data-school="Elementare">
                                <i class="fa-solid fa-child text-indigo-500"></i>
                                <span>Elementare</span>
                            </button>
                            <button type="button" onclick="selectSchool('Media')" class="school-btn text-xs py-2 px-2 rounded-lg border border-slate-200 font-medium text-slate-600 hover:bg-slate-100 transition flex flex-col items-center gap-1" data-school="Media">
                                <i class="fa-solid fa-graduation-cap text-amber-500"></i>
                                <span>Media</span>
                            </button>
                            <button type="button" onclick="selectSchool('Altro')" class="school-btn text-xs py-2 px-2 rounded-lg border border-slate-200 font-medium text-slate-600 hover:bg-slate-100 transition flex flex-col items-center gap-1" data-school="Altro">
                                <i class="fa-solid fa-building text-slate-500"></i>
                                <span>Altro</span>
                            </button>
                        </div>
                        <select id="input-school" required onchange="syncSchoolButtons()" class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-slate-800 text-sm focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white transition-all">
                            <option value="" disabled selected>Seleziona scuola...</option>
                            <option value="Elementare">Scuola Elementare (Primaria)</option>
                            <option value="Media">Scuola Media (Secondaria I°)</option>
                            <option value="Altro">Altro / Extra</option>
                        </select>
                    </div>

                    <!-- Tipo Utente Campo (Allievi, Dipendenti, Altro) -->
                    <div>
                        <label class="block text-xs font-semibold text-slate-700 uppercase tracking-wider mb-1">Tipo Utenza *</label>
                        <select id="input-type" required class="w-full px-3.5 py-2.5 bg-slate-50 border border-slate-300 rounded-xl text-slate-800 text-sm focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white transition-all">
                            <option value="" disabled selected>Seleziona tipo utente...</option>
                            <option value="Allievi">Allievi (Studenti)</option>
                            <option value="Dipendenti">Dipendenti (Docenti / Staff)</option>
                            <option value="Altro">Altro (Ospiti / Esterni)</option>
                        </select>
                    </div>

                    <!-- Numero Pasti Campo -->
                    <div>
                        <label for="input-number" class="block text-xs font-semibold text-slate-700 uppercase tracking-wider mb-1">Numero Pasti *</label>
                        <div class="relative flex items-center">
                            <button type="button" onclick="adjustNumber(-1)" class="px-3.5 py-2.5 bg-slate-100 border border-slate-300 rounded-l-xl text-slate-600 hover:bg-slate-200 font-bold">-</button>
                            <input type="number" id="input-number" min="1" step="1" value="1" required class="w-full px-3.5 py-2.5 bg-slate-50 border-y border-slate-300 text-center font-bold text-slate-900 text-base focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white transition-all">
                            <button type="button" onclick="adjustNumber(1)" class="px-3.5 py-2.5 bg-slate-100 border border-slate-300 rounded-r-xl text-slate-600 hover:bg-slate-200 font-bold">+</button>
                        </div>
                    </div>

                    <!-- Note Opzionali -->
                    <div>
                        <label for="input-notes" class="block text-xs font-semibold text-slate-700 uppercase tracking-wider mb-1">Note / Dettagli <span class="text-slate-400 font-normal">(Opzionale)</span></label>
                        <textarea id="input-notes" rows="2" placeholder="Es. Classe 2B, menu diete speciali, gita..." class="w-full px-3.5 py-2 bg-slate-50 border border-slate-300 rounded-xl text-slate-800 text-sm focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white transition-all resize-none"></textarea>
                    </div>

                    <button type="submit" id="submit-btn" class="w-full py-3 px-4 bg-sky-600 hover:bg-sky-700 active:bg-sky-800 text-white font-semibold rounded-xl shadow-md shadow-sky-200 transition-all flex items-center justify-center gap-2">
                        <i class="fa-solid fa-plus"></i>
                        <span>Registra Pasti</span>
                    </button>
                </form>
            </div>

            <!-- Right Column: Graph & REPORTS (Giornaliero, Mensile, Annuale) -->
            <div class="lg:col-span-7 space-y-6">
                
                <!-- Report fine periodo (Giornaliero / Mensile / Annuale) -->
                <div class="bg-gradient-to-br from-slate-900 to-sky-950 text-white rounded-2xl shadow-xl p-6 relative overflow-hidden">
                    <div class="flex flex-col space-y-4 border-b border-sky-800/50 pb-4">
                        <div class="flex flex-col sm:flex-row items-start sm:items-center justify-between gap-3">
                            <div>
                                <span class="inline-block px-2.5 py-0.5 rounded-full bg-sky-500/20 text-sky-300 text-xs font-semibold uppercase tracking-wider mb-1">Modulo Reporting Avanzato</span>
                                <h2 class="text-xl font-bold flex items-center gap-2">
                                    <i class="fa-solid fa-file-invoice text-sky-400"></i>
                                    Generatore Report
                                </h2>
                            </div>
                            <div class="flex items-center gap-2 w-full sm:w-auto no-print">
                                <button onclick="exportReportPDF()" class="bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-bold px-3 py-1.5 rounded-xl text-xs flex items-center gap-1.5 transition">
                                    <i class="fa-solid fa-file-pdf"></i> Esporta PDF
                                </button>
                                <button onclick="printReport()" class="bg-sky-500 hover:bg-sky-400 text-slate-950 font-bold px-3 py-1.5 rounded-xl text-xs flex items-center gap-1.5 transition">
                                    <i class="fa-solid fa-print"></i> Stampa
                                </button>
                            </div>
                        </div>

                        <!-- Selector Periodo (Giornaliero, Mensile, Annuale) -->
                        <div class="flex flex-wrap items-center justify-between gap-3 pt-2 no-print">
                            <div class="flex rounded-xl bg-slate-800/90 p-1 border border-sky-800/60">
                                <button type="button" onclick="setReportPeriod('daily')" id="period-btn-daily" class="px-3 py-1 text-xs font-semibold rounded-lg transition-all text-white bg-sky-600 shadow">Giornaliero</button>
                                <button type="button" onclick="setReportPeriod('monthly')" id="period-btn-monthly" class="px-3 py-1 text-xs font-semibold rounded-lg transition-all text-slate-400 hover:text-white">Mensile</button>
                                <button type="button" onclick="setReportPeriod('annual')" id="period-btn-annual" class="px-3 py-1 text-xs font-semibold rounded-lg transition-all text-slate-400 hover:text-white">Annuale</button>
                            </div>

                            <!-- Dynamic Inputs per Periodo -->
                            <div id="date-input-container">
                                <!-- Populated dynamically based on period selection -->
                            </div>
                        </div>
                    </div>

                    <!-- Table Summary for Selected Period -->
                    <div id="report-output" class="mt-4 space-y-4">
                        <!-- Populated dynamically -->
                    </div>
                </div>

                <!-- Visual Graph Chart -->
                <div class="bg-white rounded-2xl border border-slate-200 shadow-sm p-6 no-print">
                    <div class="flex items-center justify-between mb-4">
                        <h3 class="text-sm font-bold text-slate-800 uppercase tracking-wider flex items-center gap-2">
                            <i class="fa-solid fa-chart-bar text-sky-600"></i>
                            Ripartizione Pasti per Scuola e Tipo
                        </h3>
                    </div>
                    <div class="h-56 relative flex items-center justify-center">
                        <canvas id="summaryChart"></canvas>
                    </div>
                </div>

            </div>
        </div>

        <!-- Table & Management Section -->
        <section class="bg-white rounded-2xl border border-slate-200 shadow-sm overflow-hidden no-print">
            <div class="p-6 border-b border-slate-100 space-y-4">
                <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                    <div>
                        <h2 class="text-lg font-bold text-slate-800">Registro Storico Inserimenti</h2>
                        <p class="text-xs text-slate-500">Visualizza e gestisci le singole registrazioni giornaliere</p>
                    </div>
                    <button onclick="clearAllData()" class="px-3 py-2 text-xs font-semibold text-rose-600 bg-rose-50 hover:bg-rose-100 rounded-lg transition-colors border border-rose-200">
                        <i class="fa-solid fa-trash-can mr-1"></i> Svuota Tutto
                    </button>
                </div>

                <!-- Filters -->
                <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-3 pt-2">
                    <div class="relative">
                        <i class="fa-solid fa-magnifying-glass absolute left-3.5 top-3 text-slate-400 text-xs"></i>
                        <input type="text" id="filter-search" oninput="applyFilters()" placeholder="Cerca note..." class="w-full pl-9 pr-3.5 py-2 bg-slate-50 border border-slate-200 rounded-lg text-xs focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white">
                    </div>
                    <div>
                        <select id="filter-school" onchange="applyFilters()" class="w-full px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg text-xs focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white text-slate-700">
                            <option value="ALL">Tutte le Scuole</option>
                            <option value="Elementare">Elementare</option>
                            <option value="Media">Media</option>
                            <option value="Altro">Altro</option>
                        </select>
                    </div>
                    <div>
                        <select id="filter-type" onchange="applyFilters()" class="w-full px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg text-xs focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white text-slate-700">
                            <option value="ALL">Tutti i Tipi</option>
                            <option value="Allievi">Allievi</option>
                            <option value="Dipendenti">Dipendenti</option>
                            <option value="Altro">Altro</option>
                        </select>
                    </div>
                    <div>
                        <input type="date" id="filter-date-from" onchange="applyFilters()" class="w-full px-3 py-2 bg-slate-50 border border-slate-200 rounded-lg text-xs focus:outline-none focus:ring-2 focus:ring-sky-500 focus:bg-white text-slate-700" title="Da Data">
                    </div>
                </div>
            </div>

            <!-- Table -->
            <div class="overflow-x-auto">
                <table class="w-full text-left border-collapse">
                    <thead>
                        <tr class="bg-slate-50 text-[11px] font-bold text-slate-400 uppercase tracking-wider border-b border-slate-200">
                            <th class="py-3.5 px-6 cursor-pointer hover:text-slate-700 transition" onclick="toggleSort('date')">
                                Data <i class="fa-solid fa-sort ml-1"></i>
                            </th>
                            <th class="py-3.5 px-6">Scuola</th>
                            <th class="py-3.5 px-6">Tipo Utenza</th>
                            <th class="py-3.5 px-6 text-center cursor-pointer hover:text-slate-700 transition" onclick="toggleSort('number')">
                                N° Pasti <i class="fa-solid fa-sort ml-1"></i>
                            </th>
                            <th class="py-3.5 px-6">Note</th>
                            <th class="py-3.5 px-6 text-right">Azioni</th>
                        </tr>
                    </thead>
                    <tbody id="meals-table-body" class="divide-y divide-slate-100 text-sm">
                        <!-- Popolated dynamically -->
                    </tbody>
                </table>
            </div>

            <div id="empty-state" class="hidden p-12 text-center">
                <div class="w-16 h-16 bg-slate-100 text-slate-400 rounded-full flex items-center justify-center mx-auto mb-3 text-2xl">
                    <i class="fa-solid fa-folder-open"></i>
                </div>
                <h4 class="font-bold text-slate-700">Nessuna registrazione trovata</h4>
                <p class="text-xs text-slate-500 mt-1">Non ci sono pasti registrati corrispondenti ai criteri scelti.</p>
            </div>

            <div class="p-4 bg-slate-50 border-t border-slate-100 flex flex-col sm:flex-row justify-between items-center text-xs text-slate-500 gap-2">
                <div>
                    Mostrando <span id="filtered-count-text" class="font-bold text-slate-800">0</span> di <span id="total-count-text" class="font-bold text-slate-800">0</span> righe
                </div>
                <div class="font-semibold text-slate-700">
                    Totale pasti filtrati: <span id="filtered-sum-text" class="text-sky-600 font-bold text-sm ml-1">0</span>
                </div>
            </div>
        </section>

    </main>

    <!-- Printable Report Section (hidden on screen, visible on print) -->
    <div id="printable-area" class="hidden print-only p-8 bg-white">
        <!-- Built dynamically during print -->
    </div>

    <footer class="bg-white border-t border-slate-200 py-4 text-center text-xs text-slate-400 mt-auto no-print">
        <p>MensaTrack &copy; 2026 - Gestione Mensa Scolastica con Report Mensili.</p>
    </footer>

    <!-- JavaScript Application Logic -->
    <script>
        let meals = [];
        let sortField = 'date';
        let sortDirection = 'desc';
        let summaryChartInstance = null;
        let activeReportPeriod = 'daily'; // 'daily', 'monthly', 'annual'

        const SCHOOL_BADGES = {
            'Elementare': { bg: 'bg-indigo-100', text: 'text-indigo-800', border: 'border-indigo-200', icon: 'fa-child' },
            'Media': { bg: 'bg-amber-100', text: 'text-amber-800', border: 'border-amber-200', icon: 'fa-graduation-cap' },
            'Altro': { bg: 'bg-slate-100', text: 'text-slate-800', border: 'border-slate-200', icon: 'fa-building' }
        };

        const TYPE_BADGES = {
            'Allievi': { bg: 'bg-sky-100', text: 'text-sky-800', icon: 'fa-user-graduate' },
            'Dipendenti': { bg: 'bg-emerald-100', text: 'text-emerald-800', icon: 'fa-user-tie' },
            'Altro': { bg: 'bg-purple-100', text: 'text-purple-800', icon: 'fa-users' }
        };

        window.onload = function() {
            const today = new Date();
            const todayStr = today.toISOString().split('T')[0];

            document.getElementById('input-date').value = todayStr;

            loadMealsFromStorage();
            initChart();
            renderReportSelectorInputs();
            render();
        };

        function setReportPeriod(period) {
            activeReportPeriod = period;

            const btnDaily = document.getElementById('period-btn-daily');
            const btnMonthly = document.getElementById('period-btn-monthly');
            const btnAnnual = document.getElementById('period-btn-annual');

            [btnDaily, btnMonthly, btnAnnual].forEach(btn => {
                btn.className = "px-3 py-1 text-xs font-semibold rounded-lg transition-all text-slate-400 hover:text-white";
            });

            if (period === 'daily') btnDaily.className = "px-3 py-1 text-xs font-semibold rounded-lg transition-all text-white bg-sky-600 shadow";
            if (period === 'monthly') btnMonthly.className = "px-3 py-1 text-xs font-semibold rounded-lg transition-all text-white bg-sky-600 shadow";
            if (period === 'annual') btnAnnual.className = "px-3 py-1 text-xs font-semibold rounded-lg transition-all text-white bg-sky-600 shadow";

            renderReportSelectorInputs();
            generateReport();
        }

        function renderReportSelectorInputs() {
            const container = document.getElementById('date-input-container');
            const today = new Date();
            const todayStr = today.toISOString().split('T')[0];
            const currentMonthStr = today.toISOString().slice(0, 7);
            const currentYearStr = today.getFullYear().toString();

            if (activeReportPeriod === 'daily') {
                container.innerHTML = `
                    <input type="date" id="report-date-select" value="${todayStr}" onchange="generateReport()" class="bg-slate-800 border border-sky-700/60 rounded-xl px-3 py-1.5 text-xs font-semibold text-white focus:outline-none focus:ring-2 focus:ring-sky-400">
                `;
            } else if (activeReportPeriod === 'monthly') {
                container.innerHTML = `
                    <input type="month" id="report-month-select" value="${currentMonthStr}" onchange="generateReport()" class="bg-slate-800 border border-sky-700/60 rounded-xl px-3 py-1.5 text-xs font-semibold text-white focus:outline-none focus:ring-2 focus:ring-sky-400">
                `;
            } else if (activeReportPeriod === 'annual') {
                container.innerHTML = `
                    <input type="number" id="report-year-select" min="2020" max="2035" value="${currentYearStr}" onchange="generateReport()" class="bg-slate-800 border border-sky-700/60 rounded-xl px-3 py-1.5 text-xs font-semibold text-white focus:outline-none focus:ring-2 focus:ring-sky-400 w-24">
                `;
            }
        }

        function loadMealsFromStorage() {
            const stored = localStorage.getItem('mensatrack_data');
            if (stored) {
                try {
                    meals = JSON.parse(stored);
                } catch (e) {
                    meals = [];
                }
            } else {
                loadSampleData(false);
            }
        }

        function saveMealsToStorage() {
            localStorage.setItem('mensatrack_data', JSON.stringify(meals));
        }

        function handleFormSubmit(e) {
            e.preventDefault();

            const editId = document.getElementById('edit-id').value;
            const date = document.getElementById('input-date').value;
            const school = document.getElementById('input-school').value;
            const type = document.getElementById('input-type').value;
            const number = parseInt(document.getElementById('input-number').value, 10);
            const notes = document.getElementById('input-notes').value.trim();

            if (!date || !school || !type || isNaN(number) || number < 1) {
                showToast('Compila tutti i campi obbligatori.', 'error');
                return;
            }

            if (editId) {
                const index = meals.findIndex(m => m.id === editId);
                if (index !== -1) {
                    meals[index] = { ...meals[index], date, school, type, number, notes };
                    showToast('Registrazione aggiornata!', 'success');
                }
            } else {
                const newMeal = {
                    id: 'meal_' + Date.now() + '_' + Math.random().toString(36).substr(2, 4),
                    date,
                    school,
                    type,
                    number,
                    notes,
                    createdAt: new Date().toISOString()
                };
                meals.push(newMeal);
                showToast('Pasti registrati con successo!', 'success');
            }

            saveMealsToStorage();
            resetForm();
            render();
        }

        function resetForm() {
            document.getElementById('edit-id').value = '';
            document.getElementById('input-school').value = '';
            document.getElementById('input-type').value = '';
            document.getElementById('input-number').value = '1';
            document.getElementById('input-notes').value = '';
            
            if (!document.getElementById('input-date').value) {
                document.getElementById('input-date').value = new Date().toISOString().split('T')[0];
            }

            document.getElementById('form-title').innerText = 'Inserimento Pasti';
            document.getElementById('submit-btn').innerHTML = '<i class="fa-solid fa-plus"></i><span>Registra Pasti</span>';
            document.getElementById('cancel-edit-btn').classList.add('hidden');

            syncSchoolButtons();
        }

        function editMeal(id) {
            const meal = meals.find(m => m.id === id);
            if (!meal) return;

            document.getElementById('edit-id').value = meal.id;
            document.getElementById('input-date').value = meal.date;
            document.getElementById('input-school').value = meal.school;
            document.getElementById('input-type').value = meal.type;
            document.getElementById('input-number').value = meal.number;
            document.getElementById('input-notes').value = meal.notes || '';

            document.getElementById('form-title').innerText = 'Modifica Registrazione';
            document.getElementById('submit-btn').innerHTML = '<i class="fa-solid fa-check"></i><span>Salva Modifiche</span>';
            document.getElementById('cancel-edit-btn').classList.remove('hidden');

            syncSchoolButtons();
            document.getElementById('meal-form').scrollIntoView({ behavior: 'smooth' });
        }

        function deleteMeal(id) {
            if (confirm('Eliminare questo inserimento?')) {
                meals = meals.filter(m => m.id !== id);
                saveMealsToStorage();
                render();
                showToast('Registrazione eliminata.', 'info');
            }
        }

        function clearAllData() {
            if (meals.length === 0) return;
            if (confirm('Sei sicuro di voler cancellare TUTTI i dati?')) {
                meals = [];
                saveMealsToStorage();
                render();
                showToast('Tutti i dati eliminati.', 'info');
            }
        }

        function selectSchool(schoolName) {
            document.getElementById('input-school').value = schoolName;
            syncSchoolButtons();
        }

        function syncSchoolButtons() {
            const val = document.getElementById('input-school').value;
            document.querySelectorAll('.school-btn').forEach(btn => {
                if (btn.getAttribute('data-school') === val) {
                    btn.classList.add('bg-sky-50', 'border-sky-500', 'ring-2', 'ring-sky-200');
                    btn.classList.remove('border-slate-200');
                } else {
                    btn.classList.remove('bg-sky-50', 'border-sky-500', 'ring-2', 'ring-sky-200');
                    btn.classList.add('border-slate-200');
                }
            });
        }

        function adjustNumber(delta) {
            const input = document.getElementById('input-number');
            let val = parseInt(input.value, 10) || 1;
            val += delta;
            if (val < 1) val = 1;
            input.value = val;
        }

        function toggleSort(field) {
            if (sortField === field) {
                sortDirection = sortDirection === 'asc' ? 'desc' : 'asc';
            } else {
                sortField = field;
                sortDirection = 'desc';
            }
            renderTable();
        }

        function getFilteredAndSortedMeals() {
            const search = document.getElementById('filter-search').value.toLowerCase();
            const school = document.getElementById('filter-school').value;
            const type = document.getElementById('filter-type').value;
            const dateFrom = document.getElementById('filter-date-from').value;

            return meals.filter(meal => {
                const matchesSearch = !search || (meal.notes && meal.notes.toLowerCase().includes(search));
                const matchesSchool = school === 'ALL' || meal.school === school;
                const matchesType = type === 'ALL' || meal.type === type;
                const matchesFrom = !dateFrom || meal.date >= dateFrom;

                return matchesSearch && matchesSchool && matchesType && matchesFrom;
            }).sort((a, b) => {
                let valA = a[sortField];
                let valB = b[sortField];
                if (sortField === 'number') {
                    valA = Number(valA);
                    valB = Number(valB);
                }
                if (valA < valB) return sortDirection === 'asc' ? -1 : 1;
                if (valA > valB) return sortDirection === 'asc' ? 1 : -1;
                return 0;
            });
        }

        function applyFilters() {
            renderTable();
        }

        function render() {
            renderTable();
            renderStats();
            generateReport();
            updateChart();
        }

        function renderTable() {
            const filtered = getFilteredAndSortedMeals();
            const tbody = document.getElementById('meals-table-body');
            const emptyState = document.getElementById('empty-state');

            tbody.innerHTML = '';

            if (filtered.length === 0) {
                emptyState.classList.remove('hidden');
            } else {
                emptyState.classList.add('hidden');
                filtered.forEach(meal => {
                    const scBadge = SCHOOL_BADGES[meal.school] || SCHOOL_BADGES['Altro'];
                    const tpBadge = TYPE_BADGES[meal.type] || TYPE_BADGES['Altro'];
                    const tr = document.createElement('tr');
                    tr.className = "hover:bg-slate-50/80 transition-colors";

                    tr.innerHTML = `
                        <td class="py-3.5 px-6 font-medium text-slate-900 whitespace-nowrap">
                            <i class="fa-regular fa-calendar text-slate-400 mr-2 text-xs"></i>${formatDateString(meal.date)}
                        </td>
                        <td class="py-3.5 px-6 whitespace-nowrap">
                            <span class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-semibold ${scBadge.bg} ${scBadge.text} border ${scBadge.border}">
                                <i class="fa-solid ${scBadge.icon} text-[10px]"></i>
                                ${meal.school}
                            </span>
                        </td>
                        <td class="py-3.5 px-6 whitespace-nowrap">
                            <span class="inline-flex items-center gap-1.5 px-2.5 py-1 rounded-full text-xs font-semibold ${tpBadge.bg} ${tpBadge.text}">
                                <i class="fa-solid ${tpBadge.icon} text-[10px]"></i>
                                ${meal.type}
                            </span>
                        </td>
                        <td class="py-3.5 px-6 text-center font-bold text-slate-800">
                            <span class="inline-block bg-slate-100 text-slate-800 px-3 py-1 rounded-md text-xs font-bold">
                                ${meal.number}
                            </span>
                        </td>
                        <td class="py-3.5 px-6 text-slate-500 text-xs max-w-xs truncate">
                            ${meal.notes ? escapeHtml(meal.notes) : '<span class="text-slate-300 italic">-</span>'}
                        </td>
                        <td class="py-3.5 px-6 text-right whitespace-nowrap">
                            <button onclick="editMeal('${meal.id}')" class="p-1.5 text-slate-400 hover:text-sky-600 hover:bg-sky-50 rounded-lg transition mr-1">
                                <i class="fa-solid fa-pen"></i>
                            </button>
                            <button onclick="deleteMeal('${meal.id}')" class="p-1.5 text-slate-400 hover:text-rose-600 hover:bg-rose-50 rounded-lg transition">
                                <i class="fa-solid fa-trash-can"></i>
                            </button>
                        </td>
                    `;
                    tbody.appendChild(tr);
                });
            }

            const totalSum = filtered.reduce((acc, m) => acc + m.number, 0);
            document.getElementById('filtered-count-text').innerText = filtered.length;
            document.getElementById('total-count-text').innerText = meals.length;
            document.getElementById('filtered-sum-text').innerText = totalSum;
        }

        function renderStats() {
            const total = meals.reduce((sum, m) => sum + m.number, 0);
            const elem = meals.filter(m => m.school === 'Elementare').reduce((sum, m) => sum + m.number, 0);
            const medie = meals.filter(m => m.school === 'Media').reduce((sum, m) => sum + m.number, 0);
            const dip = meals.filter(m => m.type === 'Dipendenti').reduce((sum, m) => sum + m.number, 0);

            document.getElementById('stat-total').innerText = total;
            document.getElementById('stat-records-count').innerText = meals.length;
            document.getElementById('stat-elementari').innerText = elem;
            document.getElementById('stat-medie').innerText = medie;
            document.getElementById('stat-dipendenti').innerText = dip;
        }

        // UNIFIED REPORT GENERATION (GIORNALIERO, MENSILE, ANNUALE)
        function generateReport() {
            const reportContainer = document.getElementById('report-output');
            let filteredMeals = [];
            let periodTitle = '';
            let periodSubtitle = '';

            if (activeReportPeriod === 'daily') {
                const selectedDate = document.getElementById('report-date-select')?.value;
                if (!selectedDate) return;
                filteredMeals = meals.filter(m => m.date === selectedDate);
                periodTitle = `Report Giornaliero: ${formatDateString(selectedDate)}`;
                periodSubtitle = `Giornaliero - ${formatDateString(selectedDate)}`;
            } else if (activeReportPeriod === 'monthly') {
                const selectedMonth = document.getElementById('report-month-select')?.value; // AAAA-MM
                if (!selectedMonth) return;
                const [year, month] = selectedMonth.split('-');
                const monthName = new Date(year, month - 1).toLocaleString('it-IT', { month: 'long', year: 'numeric' });
                filteredMeals = meals.filter(m => m.date.startsWith(selectedMonth));
                periodTitle = `Report Mensile: ${monthName}`;
                periodSubtitle = `Mensile - ${monthName.toUpperCase()}`;
            } else if (activeReportPeriod === 'annual') {
                const selectedYear = document.getElementById('report-year-select')?.value;
                if (!selectedYear) return;
                filteredMeals = meals.filter(m => m.date.startsWith(selectedYear));
                periodTitle = `Report Annuale: Anno ${selectedYear}`;
                periodSubtitle = `Annuale - Anno ${selectedYear}`;
            }

            const summary = {
                'Elementare': { Allievi: 0, Dipendenti: 0, Altro: 0, Total: 0 },
                'Media': { Allievi: 0, Dipendenti: 0, Altro: 0, Total: 0 },
                'Altro': { Allievi: 0, Dipendenti: 0, Altro: 0, Total: 0 }
            };

            let grandTotal = 0;

            filteredMeals.forEach(m => {
                const sc = summary[m.school] ? m.school : 'Altro';
                const tp = summary[sc][m.type] !== undefined ? m.type : 'Altro';
                summary[sc][tp] += m.number;
                summary[sc].Total += m.number;
                grandTotal += m.number;
            });

            reportContainer.innerHTML = `
                <div class="bg-slate-800/80 rounded-xl p-4 border border-sky-800/60">
                    <div class="flex justify-between items-center mb-3">
                        <span class="text-xs uppercase font-bold tracking-wider text-sky-400">Riferimento:</span>
                        <span class="text-sm font-bold text-white capitalize">${periodTitle}</span>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-xs text-left text-slate-200 border-collapse">
                            <thead>
                                <tr class="border-b border-sky-800/80 text-sky-300 font-bold uppercase text-[10px]">
                                    <th class="py-2 px-3">Scuola</th>
                                    <th class="py-2 px-3 text-right">Allievi</th>
                                    <th class="py-2 px-3 text-right">Dipendenti</th>
                                    <th class="py-2 px-3 text-right">Altro</th>
                                    <th class="py-2 px-3 text-right text-sky-400 font-extrabold">TOTALE SCUOLA</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y divide-slate-700/50">
                                <tr>
                                    <td class="py-2.5 px-3 font-semibold text-white">Scuola Elementare</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Elementare'].Allievi}</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Elementare'].Dipendenti}</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Elementare'].Altro}</td>
                                    <td class="py-2.5 px-3 text-right font-bold text-sky-300">${summary['Elementare'].Total}</td>
                                </tr>
                                <tr>
                                    <td class="py-2.5 px-3 font-semibold text-white">Scuola Media</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Media'].Allievi}</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Media'].Dipendenti}</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Media'].Altro}</td>
                                    <td class="py-2.5 px-3 text-right font-bold text-amber-300">${summary['Media'].Total}</td>
                                </tr>
                                <tr>
                                    <td class="py-2.5 px-3 font-semibold text-white">Altro / Extra</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Altro'].Allievi}</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Altro'].Dipendenti}</td>
                                    <td class="py-2.5 px-3 text-right">${summary['Altro'].Altro}</td>
                                    <td class="py-2.5 px-3 text-right font-bold text-slate-300">${summary['Altro'].Total}</td>
                                </tr>
                            </tbody>
                            <tfoot>
                                <tr class="border-t-2 border-sky-500 bg-sky-950/60 font-bold text-white text-sm">
                                    <td class="py-3 px-3">TOTALE GENERALE PASTI</td>
                                    <td class="py-3 px-3 text-right text-sky-300">${summary['Elementare'].Allievi + summary['Media'].Allievi + summary['Altro'].Allievi}</td>
                                    <td class="py-3 px-3 text-right text-emerald-300">${summary['Elementare'].Dipendenti + summary['Media'].Dipendenti + summary['Altro'].Dipendenti}</td>
                                    <td class="py-3 px-3 text-right text-purple-300">${summary['Elementare'].Altro + summary['Media'].Altro + summary['Altro'].Altro}</td>
                                    <td class="py-3 px-3 text-right text-sky-400 font-extrabold text-base">${grandTotal}</td>
                                </tr>
                            </tfoot>
                        </table>
                    </div>
                </div>
            `;

            // Printable / PDF View Area Setup
            const printArea = document.getElementById('printable-area');
            printArea.innerHTML = `
                <div id="pdf-report-container" style="font-family: Inter, sans-serif; padding: 25px; background: white; border-radius: 8px;">
                    <div style="border-bottom: 2px solid #0284c7; padding-bottom: 12px; margin-bottom: 20px; display: flex; justify-content: space-between; align-items: flex-end;">
                        <div>
                            <h1 style="font-size: 22px; font-weight: bold; color: #0f172a; margin: 0;">REPORT EROGAZIONE PASTI SCOLASTICI</h1>
                            <p style="color: #0284c7; font-weight: bold; margin: 4px 0 0 0; font-size: 14px;">Periodo: ${periodSubtitle.toUpperCase()}</p>
                        </div>
                        <div style="text-align: right; font-size: 11px; color: #64748b;">
                            <p style="margin:0;">Generato il: ${new Date().toLocaleDateString('it-IT')} ${new Date().toLocaleTimeString('it-IT', {hour: '2-digit', minute:'2-digit'})}</p>
                            <p style="margin:0;">Sistema MensaTrack</p>
                        </div>
                    </div>

                    <table style="width: 100%; border-collapse: collapse; margin-top: 15px; font-size: 13px;">
                        <thead>
                            <tr style="background-color: #f1f5f9; text-align: left; font-size: 11px; text-transform: uppercase;">
                                <th style="padding: 10px; border: 1px solid #cbd5e1;">Scuola / Grado</th>
                                <th style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">Allievi</th>
                                <th style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">Dipendenti</th>
                                <th style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">Altro</th>
                                <th style="padding: 10px; border: 1px solid #cbd5e1; text-align: right; font-weight: bold; color: #0369a1;">TOTALE SCUOLA</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">Scuola Elementare</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Elementare'].Allievi}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Elementare'].Dipendenti}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Elementare'].Altro}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right; font-weight: bold;">${summary['Elementare'].Total}</td>
                            </tr>
                            <tr>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">Scuola Media</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Media'].Allievi}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Media'].Dipendenti}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Media'].Altro}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right; font-weight: bold;">${summary['Media'].Total}</td>
                            </tr>
                            <tr>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; font-weight: 500;">Altro / Extra</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Altro'].Allievi}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Altro'].Dipendenti}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right;">${summary['Altro'].Altro}</td>
                                <td style="padding: 10px; border: 1px solid #cbd5e1; text-align: right; font-weight: bold;">${summary['Altro'].Total}</td>
                            </tr>
                        </tbody>
                        <tfoot>
                            <tr style="background-color: #e0f2fe; font-weight: bold; font-size: 15px;">
                                <td style="padding: 12px; border: 1px solid #cbd5e1;">TOTALE GENERALE PASTI</td>
                                <td style="padding: 12px; border: 1px solid #cbd5e1; text-align: right;">${summary['Elementare'].Allievi + summary['Media'].Allievi + summary['Altro'].Allievi}</td>
                                <td style="padding: 12px; border: 1px solid #cbd5e1; text-align: right;">${summary['Elementare'].Dipendenti + summary['Media'].Dipendenti + summary['Altro'].Dipendenti}</td>
                                <td style="padding: 12px; border: 1px solid #cbd5e1; text-align: right;">${summary['Elementare'].Altro + summary['Media'].Altro + summary['Altro'].Altro}</td>
                                <td style="padding: 12px; border: 1px solid #cbd5e1; text-align: right; color: #0369a1; font-size: 16px;">${grandTotal}</td>
                            </tr>
                        </tfoot>
                    </table>
                </div>
            `;
        }

        function exportReportPDF() {
            const printArea = document.getElementById('printable-area');
            printArea.classList.remove('hidden');

            const element = document.getElementById('pdf-report-container');
            const opt = {
                margin:       10,
                filename:     `Report_Pasti_${activeReportPeriod}_${new Date().toISOString().split('T')[0]}.pdf`,
                image:        { type: 'jpeg', quality: 0.98 },
                html2canvas:  { scale: 2 },
                jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' }
            };

            showToast('Generazione PDF in corso...', 'info');

            html2pdf().set(opt).from(element).save().then(() => {
                printArea.classList.add('hidden');
                showToast('PDF Scaricato con successo!', 'success');
            }).catch(err => {
                printArea.classList.add('hidden');
                showToast('Errore durante la creazione del PDF.', 'error');
            });
        }

        function printReport() {
            window.print();
        }

        function initChart() {
            const ctx = document.getElementById('summaryChart').getContext('2d');
            summaryChartInstance = new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: ['Elementari', 'Medie', 'Altro'],
                    datasets: [
                        { label: 'Allievi', data: [0, 0, 0], backgroundColor: '#0284c7', borderRadius: 4 },
                        { label: 'Dipendenti', data: [0, 0, 0], backgroundColor: '#10b981', borderRadius: 4 },
                        { label: 'Altro', data: [0, 0, 0], backgroundColor: '#a855f7', borderRadius: 4 }
                    ]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        x: { stacked: true, grid: { display: false } },
                        y: { stacked: true, beginAtZero: true, grid: { color: '#f1f5f9' } }
                    },
                    plugins: {
                        legend: { position: 'bottom', labels: { font: { family: 'Inter', size: 11 } } }
                    }
                }
            });
        }

        function updateChart() {
            if (!summaryChartInstance) return;

            const schools = ['Elementare', 'Media', 'Altro'];
            const dataAllievi = schools.map(sc => meals.filter(m => m.school === sc && m.type === 'Allievi').reduce((sum, m) => sum + m.number, 0));
            const dataDipendenti = schools.map(sc => meals.filter(m => m.school === sc && m.type === 'Dipendenti').reduce((sum, m) => sum + m.number, 0));
            const dataAltro = schools.map(sc => meals.filter(m => m.school === sc && m.type === 'Altro').reduce((sum, m) => sum + m.number, 0));

            summaryChartInstance.data.datasets[0].data = dataAllievi;
            summaryChartInstance.data.datasets[1].data = dataDipendenti;
            summaryChartInstance.data.datasets[2].data = dataAltro;
            summaryChartInstance.update();
        }

        function exportDataCSV() {
            if (meals.length === 0) {
                showToast('Nessun dato da esportare.', 'info');
                return;
            }

            let csvContent = "data:text/csv;charset=utf-8,";
            csvContent += "ID,Data,Scuola,Tipo Utenza,Numero Pasti,Note\n";

            meals.forEach(m => {
                const row = [
                    `"${m.id}"`,
                    `"${m.date}"`,
                    `"${m.school}"`,
                    `"${m.type}"`,
                    m.number,
                    `"${(m.notes || '').replace(/"/g, '""')}"`
                ].join(",");
                csvContent += row + "\n";
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `MensaTrack_Export_${new Date().toISOString().split('T')[0]}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
            showToast('CSV scaricato!', 'success');
        }

        function triggerImport() {
            document.getElementById('importFileInput').click();
        }

        function importDataJSON(event) {
            const file = event.target.files[0];
            if (!file) return;

            const reader = new FileReader();
            reader.onload = function(e) {
                try {
                    const imported = JSON.parse(e.target.result);
                    if (Array.isArray(imported)) {
                        meals = imported;
                        saveMealsToStorage();
                        render();
                        showToast('Dati importati!', 'success');
                    }
                } catch (err) {
                    showToast('Errore lettura JSON.', 'error');
                }
            };
            reader.readAsText(file);
            event.target.value = '';
        }

        function loadSampleData(notify = true) {
            const today = new Date();
            const getDateStr = (offsetDays) => {
                const d = new Date(today);
                d.setDate(d.getDate() - offsetDays);
                return d.toISOString().split('T')[0];
            };

            meals = [
                { id: 'm1', date: getDateStr(0), school: 'Elementare', type: 'Allievi', number: 110, notes: 'Classi 1A, 2A, 3B', createdAt: new Date().toISOString() },
                { id: 'm2', date: getDateStr(0), school: 'Elementare', type: 'Dipendenti', number: 12, notes: 'Docenti accompagnatori', createdAt: new Date().toISOString() },
                { id: 'm3', date: getDateStr(0), school: 'Media', type: 'Allievi', number: 85, notes: '', createdAt: new Date().toISOString() },
                { id: 'm4', date: getDateStr(1), school: 'Media', type: 'Allievi', number: 90, notes: '', createdAt: new Date().toISOString() },
                { id: 'm5', date: getDateStr(1), school: 'Media', type: 'Dipendenti', number: 8, notes: '', createdAt: new Date().toISOString() },
                { id: 'm6', date: getDateStr(2), school: 'Elementare', type: 'Allievi', number: 105, notes: '', createdAt: new Date().toISOString() },
                { id: 'm7', date: getDateStr(2), school: 'Altro', type: 'Altro', number: 15, notes: 'Personale esterno cantiere', createdAt: new Date().toISOString() }
            ];

            saveMealsToStorage();
            render();
            if (notify) showToast('Dati dimostrativi caricati!', 'info');
        }

        function showToast(message, type = 'info') {
            const container = document.getElementById('toast-container');
            const toast = document.createElement('div');
            let bgClass = 'bg-slate-900 text-white';
            let icon = 'fa-info-circle';

            if (type === 'success') {
                bgClass = 'bg-sky-600 text-white';
                icon = 'fa-check-circle';
            } else if (type === 'error') {
                bgClass = 'bg-rose-600 text-white';
                icon = 'fa-exclamation-triangle';
            }

            toast.className = `pointer-events-auto flex items-center gap-3 px-4 py-3 rounded-xl shadow-lg ${bgClass} text-xs font-semibold transform transition-all duration-300 translate-y-2 opacity-0`;
            toast.innerHTML = `<i class="fa-solid ${icon} text-sm"></i><span>${escapeHtml(message)}</span>`;

            container.appendChild(toast);
            setTimeout(() => toast.classList.remove('translate-y-2', 'opacity-0'), 10);
            setTimeout(() => {
                toast.classList.add('opacity-0', 'translate-y-2');
                setTimeout(() => toast.remove(), 300);
            }, 3000);
        }

        function formatDateString(dateStr) {
            if (!dateStr) return '';
            const [year, month, day] = dateStr.split('-');
            return `${day}/${month}/${year}`;
        }

        function escapeHtml(str) {
            if (!str) return '';
            return str.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        }
    </script>
</body>
</html>
