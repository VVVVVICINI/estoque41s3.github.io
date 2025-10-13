<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Controle de Estoque</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jsbarcode/3.11.5/JsBarcode.all.min.js"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
        
        * {
            font-family: 'Inter', sans-serif;
        }
        
        .dark { 
            color-scheme: dark;
        }
        
        @media print {
            body * { visibility: hidden; }
            .print-area, .print-area * { visibility: visible; }
            .print-area { position: absolute; left: 0; top: 0; }
        }
        
        .gradient-bg {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        }
        
        .gradient-bg-green {
            background: linear-gradient(135deg, #10b981 0%, #059669 100%);
        }
        
        .gradient-bg-blue {
            background: linear-gradient(135deg, #3b82f6 0%, #2563eb 100%);
        }
        
        .gradient-bg-orange {
            background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
        }
        
        .card-hover {
            transition: all 0.3s ease;
        }
        
        .card-hover:hover {
            transform: translateY(-4px);
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 10px 10px -5px rgba(0, 0, 0, 0.04);
        }
        
        .btn-gradient {
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            transition: all 0.3s ease;
        }
        
        .btn-gradient:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 20px rgba(102, 126, 234, 0.4);
        }
        
        .stat-card {
            background: linear-gradient(135deg, rgba(255,255,255,0.1) 0%, rgba(255,255,255,0.05) 100%);
            backdrop-filter: blur(10px);
            border: 1px solid rgba(255,255,255,0.2);
        }
        
        .dark .stat-card {
            background: linear-gradient(135deg, rgba(255,255,255,0.05) 0%, rgba(255,255,255,0.02) 100%);
            border: 1px solid rgba(255,255,255,0.1);
        }
        
        .animate-fade-in {
            animation: fadeIn 0.5s ease-in;
        }
        
        @keyframes fadeIn {
            from { opacity: 0; transform: translateY(20px); }
            to { opacity: 1; transform: translateY(0); }
        }
        
        .badge-pulse {
            animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
        }
        
        @keyframes pulse {
            0%, 100% { opacity: 1; }
            50% { opacity: 0.7; }
        }
        
        .table-row-hover {
            transition: all 0.2s ease;
        }
        
        .table-row-hover:hover {
            background: linear-gradient(90deg, rgba(102, 126, 234, 0.05) 0%, rgba(118, 75, 162, 0.05) 100%);
        }
        
        .dark .table-row-hover:hover {
            background: linear-gradient(90deg, rgba(102, 126, 234, 0.1) 0%, rgba(118, 75, 162, 0.1) 100%);
        }
        
        .glass-effect {
            background: rgba(255, 255, 255, 0.95);
            backdrop-filter: blur(20px);
            border: 1px solid rgba(255, 255, 255, 0.3);
        }
        
        .dark .glass-effect {
            background: rgba(17, 24, 39, 0.95);
            border: 1px solid rgba(255, 255, 255, 0.1);
        }
        
        .input-modern {
            transition: all 0.3s ease;
        }
        
        .input-modern:focus {
            transform: scale(1.01);
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }
        
        .scrollbar-hide::-webkit-scrollbar {
            display: none;
        }
        
        .scrollbar-hide {
            -ms-overflow-style: none;
            scrollbar-width: none;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 via-purple-50 to-pink-50 dark:from-gray-900 dark:via-purple-900 dark:to-gray-900">
    <div id="app"></div>

    <script>
        const icons = {
            package: '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="16.5" y1="9.4" x2="7.5" y2="4.21"></line><path d="M21 16V8a2 2 0 0 0-1-1.73l-7-4a2 2 0 0 0-2 0l-7 4A2 2 0 0 0 3 8v8a2 2 0 0 0 1 1.73l7 4a2 2 0 0 0 2 0l7-4A2 2 0 0 0 21 16z"></path><polyline points="3.27 6.96 12 12.01 20.73 6.96"></polyline><line x1="12" y1="22.08" x2="12" y2="12"></line></svg>',
            plus: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="12" y1="5" x2="12" y2="19"></line><line x1="5" y1="12" x2="19" y2="12"></line></svg>',
            search: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="11" cy="11" r="8"></circle><line x1="21" y1="21" x2="16.65" y2="16.65"></line></svg>',
            edit: '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M11 4H4a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2v-7"></path><path d="M18.5 2.5a2.121 2.121 0 0 1 3 3L12 15l-4 1 1-4 9.5-9.5z"></path></svg>',
            trash: '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="3 6 5 6 21 6"></polyline><path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path></svg>',
            printer: '<svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polyline points="6 9 6 2 18 2 18 9"></polyline><path d="M6 18H4a2 2 0 0 1-2-2v-5a2 2 0 0 1 2-2h16a2 2 0 0 1 2 2v5a2 2 0 0 1-2 2h-2"></path><rect x="6" y="14" width="12" height="8"></rect></svg>',
            download: '<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="7 10 12 15 17 10"></polyline><line x1="12" y1="15" x2="12" y2="3"></line></svg>',
            upload: '<svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"></path><polyline points="17 8 12 3 7 8"></polyline><line x1="12" y1="3" x2="12" y2="15"></line></svg>',
            filter: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3"></polygon></svg>',
            tag: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20.59 13.41l-7.17 7.17a2 2 0 0 1-2.83 0L2 12V2h10l8.59 8.59a2 2 0 0 1 0 2.82z"></path><line x1="7" y1="7" x2="7.01" y2="7"></line></svg>',
            camera: '<svg xmlns="http://www.w3.org/2000/svg" width="32" height="32" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M23 19a2 2 0 0 1-2 2H3a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h4l2-3h6l2 3h4a2 2 0 0 1 2 2z"></path><circle cx="12" cy="13" r="4"></circle></svg>',
            x: '<svg xmlns="http://www.w3.org/2000/svg" width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>',
            moon: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"></path></svg>',
            sun: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="5"></circle><line x1="12" y1="1" x2="12" y2="3"></line><line x1="12" y1="21" x2="12" y2="23"></line><line x1="4.22" y1="4.22" x2="5.64" y2="5.64"></line><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"></line><line x1="1" y1="12" x2="3" y2="12"></line><line x1="21" y1="12" x2="23" y2="12"></line><line x1="4.22" y1="19.78" x2="5.64" y2="18.36"></line><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"></line></svg>',
            sparkles: '<svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M12 3v18M3 12h18M6.5 6.5l11 11M17.5 6.5l-11 11"></path></svg>'
        };

        let state = {
            items: [],
            searchTerm: '',
            filterStatus: 'todos',
            filterCategory: 'todas',
            showModal: false,
            editingItem: null,
            darkMode: false,
            selectedImage: null,
            formData: {
                codigo: '', nome: '', descricao: '', quantidade: '', status: 'venda',
                localizacao: '', observacoes: '', categoria: '', tags: '', preco: '', foto: null
            }
        };

        const categories = ['Eletrônicos', 'Eletrodomésticos', 'Informática', 'Telefonia', 'Áudio e Vídeo', 'Games', 'Acessórios', 'Outros'];

        const statusOptions = [
            { value: 'venda', label: 'Disponível para Venda', color: 'bg-gradient-to-r from-green-400 to-green-600 text-white', icon: '✓' },
            { value: 'assistencia', label: 'Assistência Técnica', color: 'bg-gradient-to-r from-yellow-400 to-orange-500 text-white', icon: '🔧' },
            { value: 'mostruario', label: 'Mostruário', color: 'bg-gradient-to-r from-blue-400 to-blue-600 text-white', icon: '👁️' },
            { value: 'uso_loja', label: 'Uso da Loja', color: 'bg-gradient-to-r from-purple-400 to-purple-600 text-white', icon: '🏪' },
            { value: 'separado', label: 'Separado p/ Entrega', color: 'bg-gradient-to-r from-orange-400 to-red-500 text-white', icon: '📦' }
        ];

        function loadData() {
            const saved = localStorage.getItem('inventoryItems');
            const savedDarkMode = localStorage.getItem('darkMode');
            if (saved) state.items = JSON.parse(saved);
            if (savedDarkMode) {
                state.darkMode = JSON.parse(savedDarkMode);
                if (state.darkMode) document.documentElement.classList.add('dark');
            }
        }

        function saveData() {
            localStorage.setItem('inventoryItems', JSON.stringify(state.items));
            localStorage.setItem('darkMode', JSON.stringify(state.darkMode));
        }

        function getStatusInfo(status) {
            return statusOptions.find(opt => opt.value === status);
        }

        function getTotalByStatus(status) {
            return state.items.filter(item => item.status === status).length;
        }

        function formatDate(dateString) {
            if (!dateString) return '-';
            return new Date(dateString).toLocaleString('pt-BR');
        }

        function getFilteredItems() {
            return state.items.filter(item => {
                const matchesSearch = 
                    item.nome.toLowerCase().includes(state.searchTerm.toLowerCase()) ||
                    item.codigo.toLowerCase().includes(state.searchTerm.toLowerCase()) ||
                    item.descricao.toLowerCase().includes(state.searchTerm.toLowerCase()) ||
                    (item.tags && item.tags.toLowerCase().includes(state.searchTerm.toLowerCase()));
                const matchesFilter = state.filterStatus === 'todos' || item.status === state.filterStatus;
                const matchesCategory = state.filterCategory === 'todas' || item.categoria === state.filterCategory;
                return matchesSearch && matchesFilter && matchesCategory;
            });
        }

        function toggleDarkMode() {
            state.darkMode = !state.darkMode;
            document.documentElement.classList.toggle('dark');
            saveData();
            render();
        }

        function openModal(item = null) {
            if (item) {
                state.editingItem = item;
                state.formData = {...item};
                state.selectedImage = item.foto;
            } else {
                state.editingItem = null;
                state.formData = { codigo: '', nome: '', descricao: '', quantidade: '', status: 'venda', localizacao: '', observacoes: '', categoria: '', tags: '', preco: '', foto: null };
                state.selectedImage = null;
            }
            state.showModal = true;
            render();
        }

        function closeModal() {
            state.showModal = false;
            state.editingItem = null;
            state.selectedImage = null;
            render();
        }

        function handleImageUpload(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onloadend = () => {
                    state.selectedImage = reader.result;
                    state.formData.foto = reader.result;
                    render();
                };
                reader.readAsDataURL(file);
            }
        }

        function removeImage() {
            state.selectedImage = null;
            state.formData.foto = null;
            render();
        }

        function handleSubmit() {
            if (!state.formData.codigo || !state.formData.nome || !state.formData.quantidade) {
                alert('Por favor, preencha os campos obrigatórios: Código, Nome e Quantidade');
                return;
            }
            const now = new Date().toISOString();
            if (state.editingItem) {
                state.items = state.items.map(item => {
                    if (item.id === state.editingItem.id) {
                        const statusChanged = item.status !== state.formData.status;
                        const newHistory = statusChanged ? [...(item.historico || []), { data: now, statusAnterior: item.status, statusNovo: state.formData.status, alteradoPor: 'Usuário' }] : item.historico || [];
                        return { ...state.formData, id: item.id, dataEntrada: item.dataEntrada, dataUltimaMovimentacao: now, historico: newHistory };
                    }
                    return item;
                });
            } else {
                state.items.push({ ...state.formData, id: Date.now(), dataEntrada: now, dataUltimaMovimentacao: now, historico: [{ data: now, statusAnterior: null, statusNovo: state.formData.status, alteradoPor: 'Usuário' }] });
            }
            saveData();
            closeModal();
        }

        function deleteItem(id) {
            if (confirm('Tem certeza que deseja excluir este item?')) {
                state.items = state.items.filter(item => item.id !== id);
                saveData();
                render();
            }
        }

        function exportToExcel() {
            const exportData = state.items.map(item => ({ 'Código': item.codigo, 'Nome': item.nome, 'Descrição': item.descricao, 'Quantidade': item.quantidade, 'Status': getStatusInfo(item.status).label, 'Categoria': item.categoria, 'Tags': item.tags, 'Preço': item.preco, 'Localização': item.localizacao, 'Data Entrada': formatDate(item.dataEntrada), 'Última Movimentação': formatDate(item.dataUltimaMovimentacao), 'Observações': item.observacoes }));
            const ws = XLSX.utils.json_to_sheet(exportData);
            const wb = XLSX.utils.book_new();
            XLSX.utils.book_append_sheet(wb, ws, 'Estoque');
            XLSX.writeFile(wb, `estoque_${new Date().toISOString().split('T')[0]}.xlsx`);
        }

        function exportToCSV() {
            const exportData = state.items.map(item => ({ 'Código': item.codigo, 'Nome': item.nome, 'Descrição': item.descricao, 'Quantidade': item.quantidade, 'Status': getStatusInfo(item.status).label, 'Categoria': item.categoria, 'Tags': item.tags, 'Preço': item.preco, 'Localização': item.localizacao, 'Data Entrada': formatDate(item.dataEntrada), 'Última Movimentação': formatDate(item.dataUltimaMovimentacao), 'Observações': item.observacoes }));
            const ws = XLSX.utils.json_to_sheet(exportData);
            const csv = XLSX.utils.sheet_to_csv(ws);
            const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement('a');
            link.href = URL.createObjectURL(blob);
            link.download = `estoque_${new Date().toISOString().split('T')[0]}.csv`;
            link.click();
        }

        function importFromFile(e) {
            const file = e.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = (event) => {
                    const data = new Uint8Array(event.target.result);
                    const workbook = XLSX.read(data, { type: 'array' });
                    const sheetName = workbook.SheetNames[0];
                    const sheet = workbook.Sheets[sheetName];
                    const jsonData = XLSX.utils.sheet_to_json(sheet);
                    const importedItems = jsonData.map((row, index) => ({ id: Date.now() + index, codigo: row['Código'] || '', nome: row['Nome'] || '', descricao: row['Descrição'] || '', quantidade: row['Quantidade'] || '', status: Object.values(statusOptions).find(s => s.label === row['Status'])?.value || 'venda', categoria: row['Categoria'] || '', tags: row['Tags'] || '', preco: row['Preço'] || '', localizacao: row['Localização'] || '', observacoes: row['Observações'] || '', dataEntrada: new Date().toISOString(), dataUltimaMovimentacao: new Date().toISOString(), historico: [], foto: null }));
                    state.items = [...state.items, ...importedItems];
                    saveData();
                    alert(`${importedItems.length} itens importados com sucesso!`);
                    render();
                };
                reader.readAsArrayBuffer(file);
            }
        }

        function printList() {
            const filteredItems = getFilteredItems();
            const printWindow = window.open('', '', 'height=600,width=800');
            printWindow.document.write('<html><head><title>Lista de Estoque</title><style>body{font-family:Inter,Arial;} table{width:100%;border-collapse:collapse;} th,td{border:1px solid #ddd;padding:8px;text-align:left;font-size:12px;} th{background:linear-gradient(135deg, #667eea 0%, #764ba2 100%);color:white;}</style></head><body><h1 style="color:#667eea;">Lista de Estoque</h1><p>Data: ' + new Date().toLocaleDateString('pt-BR') + '</p><table><tr><th>Código</th><th>Nome</th><th>Qtd</th><th>Status</th><th>Categoria</th><th>Preço</th></tr>');
            filteredItems.forEach(item => {
                printWindow.document.write(`<tr><td>${item.codigo}</td><td>${item.nome}</td><td>${item.quantidade}</td><td>${getStatusInfo(item.status).label}</td><td>${item.categoria || '-'}</td><td>${item.preco ? 'R$ ' + item.preco : '-'}</td></tr>`);
            });
            printWindow.document.write('</table></body></html>');
            printWindow.document.close();
            printWindow.print();
        }

        function generateBarcode(codigo) {
            try {
                const canvas = document.createElement('canvas');
                JsBarcode(canvas, codigo, { format: 'CODE128', displayValue: false, height: 50 });
                return canvas.toDataURL();
            } catch (e) {
                return null;
            }
        }

        function printLabel(item) {
            const barcode = generateBarcode(item.codigo);
            const printWindow = window.open('', '', 'height=400,width=600');
            printWindow.document.write('<html><head><title>Etiqueta</title><style>body{font-family:Inter,Arial;text-align:center;padding:20px;} .label{border:3px solid #667eea;padding:20px;display:inline-block;border-radius:10px;} img{margin:10px 0;}</style></head><body><div class="label"><h2 style="margin:10px 0;color:#667eea;">' + item.nome + '</h2><p style="margin:5px 0;"><strong>Código:</strong> ' + item.codigo + '</p>');
            if (barcode) printWindow.document.write('<img src="' + barcode + '" alt="Código de Barras"/>');
            printWindow.document.write('<p style="margin:5px 0;"><strong>Preço:</strong> ' + (item.preco ? 'R$ ' + item.preco : 'N/A') + '</p><p style="margin:5px 0;"><strong>Status:</strong> ' + getStatusInfo(item.status).label + '</p></div></body></html>');
            printWindow.document.close();
            printWindow.print();
        }

        function render() {
            const filteredItems = getFilteredItems();
            const isDark = state.darkMode;

            document.getElementById('app').innerHTML = `
                <div class="min-h-screen p-4 md:p-8 animate-fade-in">
                    <div class="max-w-7xl mx-auto">
                        <!-- Header Moderno -->
                        <div class="glass-effect rounded-3xl shadow-2xl p-6 md:p-8 mb-8 card-hover">
                            <div class="flex flex-col lg:flex-row items-center justify-between gap-6">
                                <div class="flex items-center gap-4">
                                    <div class="p-4 rounded-2xl gradient-bg shadow-lg">
                                        <div class="text-white">${icons.package}</div>
                                    </div>
                                    <div>
                                        <h1 class="text-3xl md:text-4xl font-bold bg-gradient-to-r from-purple-600 to-pink-600 bg-clip-text text-transparent">
                                            Controle de Estoque
                                        </h1>
                                        <p class="${isDark ? 'text-gray-400' : 'text-gray-600'} text-sm mt-1">Sistema inteligente de gerenciamento</p>
                                    </div>
                                </div>
                                
                                <div class="flex flex-wrap gap-2 justify-center">
                                    <button onclick="toggleDarkMode()" class="${isDark ? 'bg-gradient-to-r from-yellow-400 to-orange-500' : 'bg-gradient-to-r from-indigo-500 to-purple-600'} text-white px-4 py-2.5 rounded-xl hover:shadow-lg transition-all duration-300 hover:-translate-y-0.5">
                                        ${isDark ? icons.sun : icons.moon}
                                    </button>
                                    <button onclick="exportToExcel()" class="gradient-bg-green text-white px-4 py-2.5 rounded-xl hover:shadow-lg transition-all duration-300 hover:-translate-y-0.5 flex items-center gap-2 text-sm font-medium">
                                        ${icons.download} Excel
                                    </button>
                                    <button onclick="exportToCSV()" class="bg-gradient-to-r from-teal-400 to-cyan-500 text-white px-4 py-2.5 rounded-xl hover:shadow-lg transition-all duration-300 hover:-translate-y-0.5 flex items-center gap-2 text-sm font-medium">
                                        ${icons.download} CSV
                                    </button>
                                    <button onclick="document.getElementById('importFile').click()" class="bg-gradient-to-r from-purple-500 to-pink-500 text-white px-4 py-2.5 rounded-xl hover:shadow-lg transition-all duration-300 hover:-translate-y-0.5 flex items-center gap-2 text-sm font-medium">
                                        ${icons.upload} Importar
                                    </button>
                                    <input type="file" id="importFile" accept=".xlsx,.xls,.csv" onchange="importFromFile(event)" class="hidden">
                                    <button onclick="printList()" class="bg-gradient-to-r from-gray-600 to-gray-700 text-white px-4 py-2.5 rounded-xl hover:shadow-lg transition-all duration-300 hover:-translate-y-0.5 flex items-center gap-2 text-sm font-medium">
                                        ${icons.printer} Imprimir
                                    </button>
                                    <button onclick="openModal()" class="btn-gradient text-white px-6 py-2.5 rounded-xl font-semibold flex items-center gap-2 shadow-lg">
                                        ${icons.plus} Adicionar
                                    </button>
                                </div>
                            </div>
                        </div>

                        <!-- Cards de Estatísticas -->
                        <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-5 gap-4 mb-8">
                            ${statusOptions.map(opt => `
                                <div class="glass-effect rounded-2xl p-6 card-hover cursor-pointer group">
                                    <div class="flex items-center justify-between mb-3">
                                        <div class="text-3xl">${opt.icon}</div>
                                        <div class="text-3xl font-bold ${isDark ? 'text-white' : 'text-gray-800'} group-hover:scale-110 transition-transform">
                                            ${getTotalByStatus(opt.value)}
                                        </div>
                                    </div>
                                    <div class="text-sm font-medium ${isDark ? 'text-gray-300' : 'text-gray-600'} mb-2">${opt.label}</div>
                                    <div class="h-2 bg-gray-200 dark:bg-gray-700 rounded-full overflow-hidden">
                                        <div class="${opt.color.split(' ')[0]} h-full rounded-full transition-all duration-500" style="width: ${state.items.length > 0 ? (getTotalByStatus(opt.value) / state.items.length * 100) : 0}%"></div>
                                    </div>
                                </div>
                            `).join('')}
                        </div>

                        <!-- Filtros Modernos -->
                        <div class="glass-effect rounded-2xl p-6 mb-6 shadow-xl">
                            <div class="flex flex-col lg:flex-row gap-4">
                                <div class="flex-1 relative group">
                                    <div class="absolute left-4 top-1/2 -translate-y-1/2 text-purple-500 group-hover:scale-110 transition-transform">${icons.search}</div>
                                    <input type="text" placeholder="🔍 Buscar itens por código, nome, descrição ou tags..."
                                        value="${state.searchTerm}"
                                        oninput="state.searchTerm = this.value; render()"
                                        class="w-full pl-12 pr-4 py-3.5 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern text-sm font-medium">
                                </div>
                                <div class="flex gap-3">
                                    <div class="relative">
                                        <select onchange="state.filterStatus = this.value; render()" class="appearance-none px-5 py-3.5 pr-10 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 font-medium text-sm cursor-pointer hover:border-purple-400 transition-all">
                                            <option value="todos">📊 Todos Status</option>
                                            ${statusOptions.map(opt => `<option value="${opt.value}" ${state.filterStatus === opt.value ? 'selected' : ''}>${opt.icon} ${opt.label}</option>`).join('')}
                                        </select>
                                        <div class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none">${icons.filter}</div>
                                    </div>
                                    <div class="relative">
                                        <select onchange="state.filterCategory = this.value; render()" class="appearance-none px-5 py-3.5 pr-10 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 font-medium text-sm cursor-pointer hover:border-purple-400 transition-all">
                                            <option value="todas">🏷️ Todas Categorias</option>
                                            ${categories.map(cat => `<option value="${cat}" ${state.filterCategory === cat ? 'selected' : ''}>${cat}</option>`).join('')}
                                        </select>
                                        <div class="absolute right-3 top-1/2 -translate-y-1/2 pointer-events-none">${icons.tag}</div>
                                    </div>
                                </div>
                            </div>
                        </div>

                        <!-- Tabela Moderna -->
                        <div class="glass-effect rounded-2xl overflow-hidden shadow-2xl">
                            <div class="overflow-x-auto">
                                <table class="w-full">
                                    <thead>
                                        <tr class="bg-gradient-to-r from-purple-600 to-pink-600 text-white">
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Foto</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Código</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Nome</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Categoria</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Qtd</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Preço</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Status</th>
                                            <th class="px-6 py-4 text-left text-xs font-bold uppercase tracking-wider">Ações</th>
                                        </tr>
                                    </thead>
                                    <tbody class="${isDark ? 'divide-gray-700' : 'divide-gray-100'} divide-y">
                                        ${filteredItems.length === 0 ? `
                                            <tr>
                                                <td colspan="8" class="px-6 py-16 text-center">
                                                    <div class="flex flex-col items-center justify-center">
                                                        <div class="text-6xl mb-4">📦</div>
                                                        <p class="text-xl font-semibold ${isDark ? 'text-gray-300' : 'text-gray-600'} mb-2">Nenhum item encontrado</p>
                                                        <p class="text-sm ${isDark ? 'text-gray-400' : 'text-gray-500'}">Comece adicionando seus primeiros produtos!</p>
                                                    </div>
                                                </td>
                                            </tr>
                                        ` : filteredItems.map(item => {
                                            const statusInfo = getStatusInfo(item.status);
                                            return `
                                                <tr class="table-row-hover">
                                                    <td class="px-6 py-4">
                                                        ${item.foto ? 
                                                            `<img src="${item.foto}" alt="${item.nome}" class="w-16 h-16 object-cover rounded-xl shadow-md hover:scale-110 transition-transform cursor-pointer">` :
                                                            `<div class="w-16 h-16 bg-gradient-to-br from-purple-400 to-pink-400 rounded-xl flex items-center justify-center shadow-md hover:scale-110 transition-transform">
                                                                <div class="text-white text-2xl">${item.nome.charAt(0).toUpperCase()}</div>
                                                            </div>`
                                                        }
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        <div class="text-sm font-bold ${isDark ? 'text-purple-400' : 'text-purple-600'}">${item.codigo}</div>
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        <div class="text-sm font-semibold ${isDark ? 'text-white' : 'text-gray-900'}">${item.nome}</div>
                                                        ${item.tags ? `<div class="text-xs ${isDark ? 'text-gray-400' : 'text-gray-500'} mt-1">🏷️ ${item.tags}</div>` : ''}
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        ${item.categoria ? `<span class="px-3 py-1 rounded-full text-xs font-medium ${isDark ? 'bg-gray-700 text-gray-300' : 'bg-gray-100 text-gray-700'}">${item.categoria}</span>` : `<span class="${isDark ? 'text-gray-500' : 'text-gray-400'}">-</span>`}
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        <span class="px-3 py-1 rounded-lg text-sm font-bold ${isDark ? 'bg-blue-900 text-blue-200' : 'bg-blue-100 text-blue-800'}">${item.quantidade}</span>
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        <div class="text-sm font-bold ${isDark ? 'text-green-400' : 'text-green-600'}">${item.preco ? 'R$ ' + parseFloat(item.preco).toFixed(2) : '-'}</div>
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        <span class="${statusInfo.color} px-3 py-1.5 rounded-full text-xs font-bold shadow-md inline-flex items-center gap-1">
                                                            <span>${statusInfo.icon}</span>
                                                            <span>${statusInfo.label}</span>
                                                        </span>
                                                    </td>
                                                    <td class="px-6 py-4">
                                                        <div class="flex gap-2">
                                                            <button onclick='openModal(${JSON.stringify(item).replace(/'/g, "&#39;")})' class="p-2 rounded-lg ${isDark ? 'bg-blue-900 text-blue-300 hover:bg-blue-800' : 'bg-blue-100 text-blue-600 hover:bg-blue-200'} transition-all hover:scale-110" title="Editar">
                                                                ${icons.edit}
                                                            </button>
                                                            <button onclick='printLabel(${JSON.stringify(item).replace(/'/g, "&#39;")})' class="p-2 rounded-lg ${isDark ? 'bg-green-900 text-green-300 hover:bg-green-800' : 'bg-green-100 text-green-600 hover:bg-green-200'} transition-all hover:scale-110" title="Etiqueta">
                                                                ${icons.printer}
                                                            </button>
                                                            <button onclick="deleteItem(${item.id})" class="p-2 rounded-lg ${isDark ? 'bg-red-900 text-red-300 hover:bg-red-800' : 'bg-red-100 text-red-600 hover:bg-red-200'} transition-all hover:scale-110" title="Excluir">
                                                                ${icons.trash}
                                                            </button>
                                                        </div>
                                                    </td>
                                                </tr>
                                            `;
                                        }).join('')}
                                    </tbody>
                                </table>
                            </div>
                            
                            ${filteredItems.length > 0 ? `
                                <div class="${isDark ? 'bg-gray-800 border-gray-700' : 'bg-gray-50 border-gray-200'} border-t px-6 py-4">
                                    <div class="flex items-center justify-between text-sm">
                                        <div class="${isDark ? 'text-gray-400' : 'text-gray-600'}">
                                            Mostrando <span class="font-bold ${isDark ? 'text-white' : 'text-gray-900'}">${filteredItems.length}</span> de <span class="font-bold ${isDark ? 'text-white' : 'text-gray-900'}">${state.items.length}</span> itens
                                        </div>
                                        <div class="${isDark ? 'text-gray-400' : 'text-gray-600'}">
                                            Total em estoque: <span class="font-bold ${isDark ? 'text-purple-400' : 'text-purple-600'}">${state.items.reduce((sum, item) => sum + parseInt(item.quantidade || 0), 0)} unidades</span>
                                        </div>
                                    </div>
                                </div>
                            ` : ''}
                        </div>
                    </div>

                    <!-- Modal Moderno -->
                    ${state.showModal ? `
                        <div class="fixed inset-0 bg-black bg-opacity-60 backdrop-blur-sm flex items-center justify-center p-4 z-50 animate-fade-in">
                            <div class="glass-effect rounded-3xl shadow-2xl max-w-5xl w-full max-h-[90vh] overflow-hidden">
                                <div class="gradient-bg px-8 py-6 flex items-center justify-between">
                                    <div class="flex items-center gap-3">
                                        <div class="text-white text-2xl">${state.editingItem ? '✏️' : '✨'}</div>
                                        <h2 class="text-2xl font-bold text-white">
                                            ${state.editingItem ? 'Editar Item' : 'Adicionar Novo Item'}
                                        </h2>
                                    </div>
                                    <button onclick="closeModal()" class="text-white hover:bg-white hover:bg-opacity-20 p-2 rounded-xl transition-all">
                                        ${icons.x}
                                    </button>
                                </div>

                                <div class="p-8 overflow-y-auto max-h-[calc(90vh-100px)] scrollbar-hide">
                                    <!-- Upload de Foto -->
                                    <div class="mb-8">
                                        <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-3">📸 Foto do Produto</label>
                                        <div class="flex items-center gap-6">
                                            ${state.selectedImage ? `
                                                <div class="relative group">
                                                    <img src="${state.selectedImage}" alt="Preview" class="w-32 h-32 object-cover rounded-2xl shadow-xl ring-4 ring-purple-200">
                                                    <button onclick="removeImage()" class="absolute -top-2 -right-2 bg-gradient-to-r from-red-500 to-pink-500 text-white rounded-full p-2 shadow-lg hover:scale-110 transition-transform">
                                                        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>
                                                    </button>
                                                </div>
                                            ` : `
                                                <div class="w-32 h-32 bg-gradient-to-br from-purple-100 to-pink-100 dark:from-purple-900 dark:to-pink-900 rounded-2xl flex items-center justify-center cursor-pointer hover:scale-105 transition-transform shadow-lg" onclick="document.getElementById('photoInput').click()">
                                                    <div class="text-purple-400">${icons.camera}</div>
                                                </div>
                                            `}
                                            <button onclick="document.getElementById('photoInput').click()" class="btn-gradient text-white px-6 py-3 rounded-xl font-semibold shadow-lg">
                                                Escolher Foto
                                            </button>
                                            <input type="file" id="photoInput" accept="image/*" onchange="handleImageUpload(event)" class="hidden">
                                        </div>
                                    </div>

                                    <!-- Formulário em Grid -->
                                    <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-6">
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Código *</label>
                                            <input type="text" value="${state.formData.codigo}" 
                                                oninput="state.formData.codigo = this.value"
                                                class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern font-medium"
                                                placeholder="Ex: PROD001">
                                        </div>
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Quantidade *</label>
                                            <input type="number" value="${state.formData.quantidade}"
                                                oninput="state.formData.quantidade = this.value"
                                                class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern font-medium"
                                                placeholder="0">
                                        </div>
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Preço (R$)</label>
                                            <input type="number" step="0.01" value="${state.formData.preco}"
                                                oninput="state.formData.preco = this.value"
                                                class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern font-medium"
                                                placeholder="0.00">
                                        </div>
                                    </div>

                                    <div class="mb-6">
                                        <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Nome do Produto *</label>
                                        <input type="text" value="${state.formData.nome}"
                                            oninput="state.formData.nome = this.value"
                                            class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern font-medium"
                                            placeholder="Digite o nome do produto">
                                    </div>

                                    <div class="mb-6">
                                        <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Descrição</label>
                                        <input type="text" value="${state.formData.descricao}"
                                            oninput="state.formData.descricao = this.value"
                                            class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern"
                                            placeholder="Descreva o produto">
                                    </div>

                                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Categoria</label>
                                            <select onchange="state.formData.categoria = this.value" class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 font-medium cursor-pointer">
                                                <option value="">Selecione uma categoria</option>
                                                ${categories.map(cat => `<option value="${cat}" ${state.formData.categoria === cat ? 'selected' : ''}>${cat}</option>`).join('')}
                                            </select>
                                        </div>
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Status *</label>
                                            <select onchange="state.formData.status = this.value" class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 font-medium cursor-pointer">
                                                ${statusOptions.map(opt => `<option value="${opt.value}" ${state.formData.status === opt.value ? 'selected' : ''}>${opt.icon} ${opt.label}</option>`).join('')}
                                            </select>
                                        </div>
                                    </div>

                                    <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Localização</label>
                                            <input type="text" value="${state.formData.localizacao}" placeholder="Ex: Prateleira A1"
                                                oninput="state.formData.localizacao = this.value"
                                                class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern">
                                        </div>
                                        <div>
                                            <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Tags</label>
                                            <input type="text" value="${state.formData.tags}" placeholder="Ex: novo, promoção, destaque"
                                                oninput="state.formData.tags = this.value"
                                                class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all input-modern">
                                        </div>
                                    </div>

                                    <div class="mb-6">
                                        <label class="block text-sm font-bold ${isDark ? 'text-gray-300' : 'text-gray-700'} mb-2">Observações</label>
                                        <textarea rows="3" oninput="state.formData.observacoes = this.value"
                                            class="w-full px-4 py-3 border-2 ${isDark ? 'bg-gray-800 border-gray-700 text-white' : 'bg-white border-gray-200'} rounded-xl focus:ring-4 focus:ring-purple-200 focus:border-purple-400 transition-all resize-none"
                                            placeholder="Adicione observações sobre o produto">${state.formData.observacoes}</textarea>
                                    </div>

                                    ${state.editingItem && state.editingItem.historico && state.editingItem.historico.length > 0 ? `
                                        <div class="mb-6 ${isDark ? 'bg-gray-800' : 'bg-purple-50'} rounded-2xl p-6">
                                            <h3 class="text-lg font-bold ${isDark ? 'text-white' : 'text-gray-800'} mb-4 flex items-center gap-2">
                                                📜 Histórico de Alterações
                                            </h3>
                                            <div class="space-y-3 max-h-48 overflow-y-auto scrollbar-hide">
                                                ${state.editingItem.historico.map(hist => `
                                                    <div class="${isDark ? 'bg-gray-700' : 'bg-white'} rounded-xl p-4 shadow">
                                                        <div class="text-xs ${isDark ? 'text-gray-400' : 'text-gray-500'} mb-1">${formatDate(hist.data)}</div>
                                                        <div class="text-sm ${isDark ? 'text-white' : 'text-gray-800'} font-medium">
                                                            ${hist.statusAnterior ? 
                                                                `Status alterado de <span class="text-purple-600 font-bold">${getStatusInfo(hist.statusAnterior).label}</span> para <span class="text-purple-600 font-bold">${getStatusInfo(hist.statusNovo).label}</span>` :
                                                                `Item criado com status <span class="text-purple-600 font-bold">${getStatusInfo(hist.statusNovo).label}</span>`
                                                            }
                                                        </div>
                                                    </div>
                                                `).join('')}
                                            </div>
                                        </div>
                                    ` : ''}

                                    ${state.editingItem ? `
                                        <div class="grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
                                            <div class="${isDark ? 'bg-gray-800' : 'bg-blue-50'} p-4 rounded-xl">
                                                <div class="text-xs font-bold ${isDark ? 'text-gray-400' : 'text-gray-600'} mb-1">📅 Data de Entrada</div>
                                                <div class="text-sm font-semibold ${isDark ? 'text-white' : 'text-gray-800'}">${formatDate(state.editingItem.dataEntrada)}</div>
                                            </div>
                                            <div class="${isDark ? 'bg-gray-800' : 'bg-green-50'} p-4 rounded-xl">
                                                <div class="text-xs font-bold ${isDark ? 'text-gray-400' : 'text-gray-600'} mb-1">🔄 Última Movimentação</div>
                                                <div class="text-sm font-semibold ${isDark ? 'text-white' : 'text-gray-800'}">${formatDate(state.editingItem.dataUltimaMovimentacao)}</div>
                                            </div>
                                        </div>
                                    ` : ''}

                                    <div class="flex gap-4 justify-end pt-4 border-t ${isDark ? 'border-gray-700' : 'border-gray-200'}">
                                        <button onclick="closeModal()" class="px-6 py-3 border-2 ${isDark ? 'border-gray-600 text-gray-300 hover:bg-gray-800' : 'border-gray-300 text-gray-700 hover:bg-gray-50'} rounded-xl font-semibold transition-all hover:scale-105">
                                            Cancelar
                                        </button>
                                        <button onclick="handleSubmit()" class="btn-gradient text-white px-8 py-3 rounded-xl font-bold shadow-lg flex items-center gap-2">
                                            ${state.editingItem ? '💾 Salvar Alterações' : '✨ Adicionar Item'}
                                        </button>
                                    </div>
                                </div>
                            </div>
                        </div>
                    ` : ''}
                </div>
            `;
        }

        loadData();
        render();
    </script>
</body>
</html>
