<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistem Manajemen Alat Rumah Sakit</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.5);
            z-index: 1000;
            overflow-y: auto;
        }
        .modal.active {
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 1rem;
        }
        .notification {
            position: fixed;
            top: 1rem;
            right: 1rem;
            padding: 1rem 1.5rem;
            border-radius: 0.5rem;
            color: white;
            z-index: 2000;
            display: none;
            animation: slideIn 0.3s ease;
        }
        .notification.active {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        .notification.success {
            background: #10b981;
        }
        .notification.error {
            background: #ef4444;
        }
        @keyframes slideIn {
            from {
                transform: translateX(100%);
                opacity: 0;
            }
            to {
                transform: translateX(0);
                opacity: 1;
            }
        }
        .qr-code {
            width: 80px;
            height: 80px;
            border: 3px solid #1f2937;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 3rem;
            background: white;
            border-radius: 0.5rem;
        }
    </style>
</head>
<body class="bg-gradient-to-br from-blue-50 to-indigo-100 min-h-screen">
    
    <!-- Notification -->
    <div id="notification" class="notification">
        <span id="notificationIcon">✓</span>
        <span id="notificationText"></span>
    </div>

    <div class="container mx-auto px-4 py-8">
        <!-- Header -->
        <div class="bg-white rounded-2xl shadow-xl p-6 mb-6">
            <h1 class="text-3xl font-bold text-gray-800 mb-2">Sistem Manajemen Alat Rumah Sakit</h1>
            <p class="text-gray-600">Kelola alat medis dengan sistem QR Code</p>
        </div>

        <!-- Add Button -->
        <button onclick="openAddModal()" class="w-full bg-blue-600 text-white py-4 rounded-xl hover:bg-blue-700 font-semibold flex items-center justify-center gap-2 shadow-lg mb-6">
            <span style="font-size: 1.5rem;">+</span>
            Tambah Alat Baru
        </button>

        <!-- Search -->
        <div class="mb-6">
            <input 
                type="text" 
                id="searchInput" 
                placeholder="Cari alat, SN, atau ruangan..." 
                class="w-full pl-12 pr-4 py-3 border rounded-xl focus:ring-2 focus:ring-blue-500 bg-white shadow"
                oninput="filterEquipment()"
            >
        </div>

        <!-- Equipment Grid -->
        <div id="equipmentGrid" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
            <!-- Equipment cards will be inserted here -->
        </div>

        <!-- Empty State -->
        <div id="emptyState" class="text-center py-16 hidden">
            <p class="text-gray-500 text-lg">Belum ada alat. Tambahkan alat pertama Anda!</p>
        </div>
    </div>

    <!-- Modal Add Equipment -->
    <div id="addModal" class="modal">
        <div class="bg-white rounded-lg p-6 w-full max-w-md">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-gray-800">Tambah Alat Baru</h2>
                <button onclick="closeModal('addModal')" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div class="space-y-4">
                <input type="text" id="namaAlat" placeholder="Nama Alat" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <input type="text" id="typeAlat" placeholder="Type Alat" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <input type="text" id="snAlat" placeholder="Serial Number (SN)" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <input type="text" id="ruangan" placeholder="Ruangan" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
            </div>
            <button onclick="addEquipment()" class="w-full mt-6 bg-blue-600 text-white py-3 rounded-lg hover:bg-blue-700 font-semibold">
                Simpan
            </button>
        </div>
    </div>

    <!-- Modal Kalibrasi -->
    <div id="kalibrasiModal" class="modal">
        <div class="bg-white rounded-lg p-6 w-full max-w-md">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-gray-800">Kalibrasi</h2>
                <button onclick="closeModal('kalibrasiModal')" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div id="kalibrasiEquipmentInfo" class="mb-4 p-3 bg-gray-100 rounded text-sm"></div>
            <div class="space-y-4">
                <input type="text" id="kalNamaPerusahaan" placeholder="Nama Perusahaan" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <select id="kalLaikPakai" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                    <option value="laik">Laik Pakai</option>
                    <option value="tidak_laik">Tidak Laik Pakai</option>
                </select>
                <input type="date" id="kalTanggal" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <input type="text" id="kalTeknisi" placeholder="Nama Teknisi" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
            </div>
            <button onclick="saveKalibrasi()" class="w-full mt-6 bg-green-600 text-white py-3 rounded-lg hover:bg-green-700 font-semibold">
                Simpan Kalibrasi
            </button>
        </div>
    </div>

    <!-- Modal Perbaikan -->
    <div id="perbaikanModal" class="modal">
        <div class="bg-white rounded-lg p-6 w-full max-w-md max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-gray-800">Perbaikan & Pemeliharaan</h2>
                <button onclick="closeModal('perbaikanModal')" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div id="perbaikanEquipmentInfo" class="mb-4 p-3 bg-gray-100 rounded text-sm"></div>
            <div class="space-y-4">
                <input type="text" id="perbNamaPerusahaan" placeholder="Nama Perusahaan" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <select id="perbJenis" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500" onchange="togglePerbaikanFields()">
                    <option value="perbaikan">Perbaikan</option>
                    <option value="pemeliharaan">Pemeliharaan</option>
                </select>
                <textarea id="perbPenyebab" placeholder="Penyebab" rows="3" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"></textarea>
                <input type="text" id="perbTeknisi" placeholder="Nama Teknisi" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <select id="perbStatus" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                    <option value="ya">Sudah Diperbaiki</option>
                    <option value="tidak">Belum Diperbaiki</option>
                </select>
                <textarea id="perbTroubleshooting" placeholder="Troubleshooting" rows="3" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"></textarea>
            </div>
            <button onclick="savePerbaikan()" class="w-full mt-6 bg-orange-600 text-white py-3 rounded-lg hover:bg-orange-700 font-semibold">
                Simpan Data
            </button>
        </div>
    </div>

    <!-- Modal Laporan -->
    <div id="laporanModal" class="modal">
        <div class="bg-white rounded-lg p-6 w-full max-w-md">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-gray-800">Laporan</h2>
                <button onclick="closeModal('laporanModal')" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div id="laporanEquipmentInfo" class="mb-4 p-3 bg-gray-100 rounded text-sm"></div>
            <div class="space-y-4">
                <input type="text" id="lapNamaPelapor" placeholder="Nama Pelapor" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500">
                <textarea id="lapIsiLaporan" placeholder="Isi Laporan" rows="4" class="w-full px-4 py-2 border rounded-lg focus:ring-2 focus:ring-blue-500"></textarea>
            </div>
            <button onclick="saveLaporan()" class="w-full mt-6 bg-red-600 text-white py-3 rounded-lg hover:bg-red-700 font-semibold">
                Kirim Laporan ke PIC
            </button>
        </div>
    </div>

    <!-- Modal Record -->
    <div id="recordModal" class="modal">
        <div class="bg-white rounded-lg p-6 w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <div class="flex justify-between items-center mb-4">
                <h2 class="text-xl font-bold text-gray-800">Record History</h2>
                <button onclick="closeModal('recordModal')" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <div id="recordEquipmentInfo" class="mb-4 p-3 bg-gray-100 rounded text-sm"></div>
            <div id="recordList" class="space-y-4"></div>
        </div>
    </div>

    <!-- Modal Delete Confirm -->
    <div id="deleteModal" class="modal">
        <div class="bg-white rounded-lg p-6 w-full max-w-md">
            <div class="flex items-center gap-3 mb-4">
                <div class="p-3 bg-red-100 rounded-full">
                    <span class="text-red-600 text-2xl">🗑️</span>
                </div>
                <h2 class="text-xl font-bold text-gray-800">Hapus Alat</h2>
            </div>
            <p class="text-gray-700 mb-2">Apakah Anda yakin ingin menghapus alat ini?</p>
            <div id="deleteEquipmentInfo" class="bg-gray-100 rounded p-3 mb-6"></div>
            <div class="flex gap-3">
                <button onclick="closeModal('deleteModal')" class="flex-1 px-4 py-2 border border-gray-300 rounded-lg hover:bg-gray-50 font-semibold">
                    Batal
                </button>
                <button onclick="confirmDelete()" class="flex-1 px-4 py-2 bg-red-600 text-white rounded-lg hover:bg-red-700 font-semibold">
                    Hapus
                </button>
            </div>
        </div>
    </div>

    <script>
        let equipment = [];
        let currentEquipmentId = null;
        let equipmentToDelete = null;

        // Load data from localStorage
        function loadData() {
            const data = localStorage.getItem('hospital_equipment');
            if (data) {
                equipment = JSON.parse(data);
            }
            renderEquipment();
        }

        // Save data to localStorage
        function saveData() {
            localStorage.setItem('hospital_equipment', JSON.stringify(equipment));
        }

        // Show notification
        function showNotification(message, type = 'success') {
            const notification = document.getElementById('notification');
            const notificationText = document.getElementById('notificationText');
            const notificationIcon = document.getElementById('notificationIcon');
            
            notification.className = `notification ${type} active`;
            notificationText.textContent = message;
            notificationIcon.textContent = type === 'success' ? '✓' : '✕';
            
            setTimeout(() => {
                notification.classList.remove('active');
            }, 3000);
        }

        // Open/Close Modal
        function openModal(modalId) {
            document.getElementById(modalId).classList.add('active');
        }

        function closeModal(modalId) {
            document.getElementById(modalId).classList.remove('active');
            currentEquipmentId = null;
        }

        function openAddModal() {
            document.getElementById('namaAlat').value = '';
            document.getElementById('typeAlat').value = '';
            document.getElementById('snAlat').value = '';
            document.getElementById('ruangan').value = '';
            openModal('addModal');
        }

        // Add Equipment
        function addEquipment() {
            const namaAlat = document.getElementById('namaAlat').value.trim();
            const typeAlat = document.getElementById('typeAlat').value.trim();
            const snAlat = document.getElementById('snAlat').value.trim();
            const ruangan = document.getElementById('ruangan').value.trim();

            if (!namaAlat || !typeAlat || !snAlat || !ruangan) {
                showNotification('Mohon lengkapi semua field', 'error');
                return;
            }

            const newEquipment = {
                id: Date.now().toString(),
                namaAlat,
                typeAlat,
                snAlat,
                ruangan,
                records: [],
                createdAt: new Date().toISOString()
            };

            equipment.push(newEquipment);
            saveData();
            renderEquipment();
            closeModal('addModal');
            showNotification('Alat berhasil ditambahkan');
        }

        // Open Kalibrasi Modal
        function openKalibrasiModal(equipmentId) {
            currentEquipmentId = equipmentId;
            const eq = equipment.find(e => e.id === equipmentId);
            
            document.getElementById('kalibrasiEquipmentInfo').innerHTML = `
                <p class="text-gray-600">Alat: <span class="font-semibold">${eq.namaAlat}</span></p>
                <p class="text-gray-600">SN: ${eq.snAlat}</p>
            `;
            
            document.getElementById('kalNamaPerusahaan').value = '';
            document.getElementById('kalLaikPakai').value = 'laik';
            document.getElementById('kalTanggal').value = '';
            document.getElementById('kalTeknisi').value = '';
            
            openModal('kalibrasiModal');
        }

        // Save Kalibrasi
        function saveKalibrasi() {
            const namaPerusahaan = document.getElementById('kalNamaPerusahaan').value.trim();
            const laikPakai = document.getElementById('kalLaikPakai').value;
            const tanggal = document.getElementById('kalTanggal').value;
            const teknisi = document.getElementById('kalTeknisi').value.trim();

            if (!namaPerusahaan || !tanggal || !teknisi) {
                showNotification('Mohon lengkapi semua field', 'error');
                return;
            }

            const record = {
                id: Date.now().toString(),
                type: 'kalibrasi',
                namaPerusahaan,
                laikPakai,
                tanggalPengerjaan: tanggal,
                teknisi,
                timestamp: new Date().toISOString()
            };

            const eq = equipment.find(e => e.id === currentEquipmentId);
            eq.records.push(record);
            saveData();
            renderEquipment();
            closeModal('kalibrasiModal');
            showNotification('Data kalibrasi berhasil disimpan');
        }

        // Open Perbaikan Modal
        function openPerbaikanModal(equipmentId) {
            currentEquipmentId = equipmentId;
            const eq = equipment.find(e => e.id === equipmentId);
            
            document.getElementById('perbaikanEquipmentInfo').innerHTML = `
                <p class="text-gray-600">Alat: <span class="font-semibold">${eq.namaAlat}</span></p>
                <p class="text-gray-600">SN: ${eq.snAlat}</p>
            `;
            
            document.getElementById('perbNamaPerusahaan').value = '';
            document.getElementById('perbJenis').value = 'perbaikan';
            document.getElementById('perbPenyebab').value = '';
            document.getElementById('perbTeknisi').value = '';
            document.getElementById('perbStatus').value = 'ya';
            document.getElementById('perbTroubleshooting').value = '';
            
            togglePerbaikanFields();
            openModal('perbaikanModal');
        }

        function togglePerbaikanFields() {
            const jenis = document.getElementById('perbJenis').value;
            const statusField = document.getElementById('perbStatus');
            statusField.style.display = jenis === 'perbaikan' ? 'block' : 'none';
        }

        // Save Perbaikan
        function savePerbaikan() {
            const namaPerusahaan = document.getElementById('perbNamaPerusahaan').value.trim();
            const jenis = document.getElementById('perbJenis').value;
            const penyebab = document.getElementById('perbPenyebab').value.trim();
            const teknisi = document.getElementById('perbTeknisi').value.trim();
            const sudahDiperbaiki = document.getElementById('perbStatus').value;
            const troubleshooting = document.getElementById('perbTroubleshooting').value.trim();

            if (!namaPerusahaan || !penyebab || !teknisi || !troubleshooting) {
                showNotification('Mohon lengkapi semua field', 'error');
                return;
            }

            const record = {
                id: Date.now().toString(),
                type: 'perbaikan_pemeliharaan',
                namaPerusahaan,
                jenis,
                penyebab,
                teknisi,
                sudahDiperbaiki: jenis === 'perbaikan' ? sudahDiperbaiki : null,
                troubleshooting,
                timestamp: new Date().toISOString()
            };

            const eq = equipment.find(e => e.id === currentEquipmentId);
            eq.records.push(record);
            saveData();
            renderEquipment();
            closeModal('perbaikanModal');
            showNotification('Data perbaikan/pemeliharaan berhasil disimpan');
        }

        // Open Laporan Modal
        function openLaporanModal(equipmentId) {
            currentEquipmentId = equipmentId;
            const eq = equipment.find(e => e.id === equipmentId);
            
            document.getElementById('laporanEquipmentInfo').innerHTML = `
                <p class="text-gray-600">Alat: <span class="font-semibold">${eq.namaAlat}</span></p>
                <p class="text-gray-600">SN: ${eq.snAlat}</p>
            `;
            
            document.getElementById('lapNamaPelapor').value = '';
            document.getElementById('lapIsiLaporan').value = '';
            
            openModal('laporanModal');
        }

        // Save Laporan
        function saveLaporan() {
            const namaPelapor = document.getElementById('lapNamaPelapor').value.trim();
            const laporan = document.getElementById('lapIsiLaporan').value.trim();

            if (!namaPelapor || !laporan) {
                showNotification('Mohon lengkapi semua field', 'error');
                return;
            }

            const record = {
                id: Date.now().toString(),
                type: 'laporan',
                namaPelapor,
                laporan,
                timestamp: new Date().toISOString()
            };

            const eq = equipment.find(e => e.id === currentEquipmentId);
            eq.records.push(record);
            saveData();
            renderEquipment();
            closeModal('laporanModal');
            showNotification('Laporan telah dikirim ke PIC Elektromedis');
        }

        // Open Record Modal
        function openRecordModal(equipmentId) {
            const eq = equipment.find(e => e.id === equipmentId);
            
            document.getElementById('recordEquipmentInfo').innerHTML = `
                <p class="text-gray-600">Alat: <span class="font-semibold">${eq.namaAlat}</span></p>
                <p class="text-gray-600">SN: ${eq.snAlat}</p>
                <p class="text-gray-600">Ruangan: ${eq.ruangan}</p>
            `;
            
            const recordList = document.getElementById('recordList');
            
            if (eq.records.length === 0) {
                recordList.innerHTML = '<p class="text-center text-gray-500 py-8">Belum ada record</p>';
            } else {
                recordList.innerHTML = eq.records.map(record => {
                    const date = new Date(record.timestamp).toLocaleDateString('id-ID', {
                        year: 'numeric',
                        month: 'long',
                        day: 'numeric',
                        hour: '2-digit',
                        minute: '2-digit'
                    });
                    
                    let badgeClass = '';
                    let badgeText = '';
                    let details = '';
                    
                    if (record.type === 'kalibrasi') {
                        badgeClass = 'bg-green-100 text-green-800';
                        badgeText = 'Kalibrasi';
                        details = `
                            <p><span class="font-semibold">Perusahaan:</span> ${record.namaPerusahaan}</p>
                            <p><span class="font-semibold">Status:</span> ${record.laikPakai === 'laik' ? 'Laik Pakai' : 'Tidak Laik Pakai'}</p>
                            <p><span class="font-semibold">Tanggal:</span> ${record.tanggalPengerjaan}</p>
                            <p><span class="font-semibold">Teknisi:</span> ${record.teknisi}</p>
                        `;
                    } else if (record.type === 'perbaikan_pemeliharaan') {
                        badgeClass = 'bg-orange-100 text-orange-800';
                        badgeText = 'Perbaikan/Pemeliharaan';
                        details = `
                            <p><span class="font-semibold">Perusahaan:</span> ${record.namaPerusahaan}</p>
                            <p><span class="font-semibold">Jenis:</span> ${record.jenis === 'perbaikan' ? 'Perbaikan' : 'Pemeliharaan'}</p>
                            <p><span class="font-semibold">Penyebab:</span> ${record.penyebab}</p>
                            <p><span class="font-semibold">Teknisi:</span> ${record.teknisi}</p>
                            ${record.jenis === 'perbaikan' ? `<p><span class="font-semibold">Status:</span> ${record.sudahDiperbaiki === 'ya' ? 'Sudah Diperbaiki' : 'Belum Diperbaiki'}</p>` : ''}
                            <p><span class="font-semibold">Troubleshooting:</span> ${record.troubleshooting}</p>
                        `;
                    } else {
                        badgeClass = 'bg-red-100 text-red-800';
                        badgeText = 'Laporan';
                        details = `
                            <p><span class="font-semibold">Pelapor:</span> ${record.namaPelapor}</p>
                            <p><span class="font-semibold">Laporan:</span> ${record.laporan}</p>
                        `;
                    }
                    
                    return `
                        <div class="border rounded-lg p-4">
                            <div class="flex justify-between items-start mb-2">
                                <span class="px-3 py-1 rounded-full text-xs font-semibold ${badgeClass}">${badgeText}</span>
                                <span class="text-xs text-gray-500">${date}</span>
                            </div>
                            <div class="space-y-1 text-sm">${details}</div>
                        </div>
                    `;
                }).join('');
            }
            
            openModal('recordModal');
        }

        // Delete Equipment
        function openDeleteModal(equipmentId) {
            equipmentToDelete = equipmentId;
            const eq = equipment.find(e => e.id === equipmentId);
            
            document.getElementById('deleteEquipmentInfo').innerHTML = `
                <p class="font-semibold text-gray-800">${eq.namaAlat}</p>
                <p class="text-sm text-gray-600">SN: ${eq.snAlat}</p>
                <p class="text-sm text-gray-600">Ruangan: ${eq.ruangan}</p>
                <p class="text-sm text-red-600 mt-2">⚠️ Semua record akan ikut terhapus (${eq.records.length} record)</p>
            `;
            
            openModal('deleteModal');
        }

        function confirmDelete() {
            equipment = equipment.filter(e => e.id !== equipmentToDelete);
            saveData();
            renderEquipment();
            closeModal('deleteModal');
            showNotification('Alat berhasil dihapus');
            equipmentToDelete = null;
        }

        // Filter Equipment
        function filterEquipment() {
            renderEquipment();
        }

        // Render Equipment
        function renderEquipment() {
            const searchTerm = document.getElementById('searchInput').value.toLowerCase();
            const filtered = equipment.filter(eq => 
                eq.namaAlat.toLowerCase().includes(searchTerm) ||
                eq.snAlat.toLowerCase().includes(searchTerm) ||
                eq.ruangan.toLowerCase().includes(searchTerm)
            );

            const grid = document.getElementById('equipmentGrid');
            const emptyState = document.getElementById('emptyState');

            if (filtered.length === 0) {
                grid.innerHTML = '';
                emptyState.classList.remove('hidden');
                return;
            }

            emptyState.classList.add('hidden');
            grid.innerHTML = filtered.map(eq => `
                <div class="bg-white rounded-xl shadow-lg p-6 hover:shadow-xl transition-shadow relative">
                    <button onclick="openDeleteModal('${eq.id}')" class="absolute top-4 right-4 text-red-500 hover:text-red-700 hover:bg-red-50 p-2 rounded-lg transition-colors" title="Hapus Alat">
                        🗑️
                    </button>
                    
                    <div class="flex justify-between items-start mb-4 pr-8">
                        <div>
                            <h3 class="text-lg font-bold text-gray-800">${eq.namaAlat}</h3>
                            <p class="text-sm text-gray-600">Type: ${eq.typeAlat}</p>
                            <p class="text-sm text-gray-600">SN: ${eq.snAlat}</p>
                            <p class="text-sm text-gray-600">Ruangan: ${eq.ruangan}</p>
                        </div>
                        <div class="qr-code">⊞</div>
                    </div>
                    
                    <div class="grid grid-cols-2 gap-2 mt-4">
                        <button onclick="openKalibrasiModal('${eq.id}')" class="bg-green-500 text-white py-2 px-3 rounded-lg hover:bg-green-600 text-sm font-semibold">
                            Kalibrasi
                        </button>
                        <button onclick="openPerbaikanModal('${eq.id}')" class="bg-orange-500 text-white py-2 px-3 rounded-lg hover:bg-orange-600 text-sm font-semibold">
                            Perbaikan
                        </button>
                        <button onclick="openLaporanModal('${eq.id}')" class="bg-red-500 text-white py-2 px-3 rounded-lg hover:bg-red-600 text-sm font-semibold">
                            Laporan
                        </button>
                        <button onclick="openRecordModal('${eq.id}')" class="bg-blue-500 text-white py-2 px-3 rounded-lg hover:bg-blue-600 text-sm font-semibold">
                            Record
                        </button>
                    </div>
                    
                    <div class="mt-3 text-xs text-gray-500 text-center">
                        ${eq.records.length} record tersimpan
                    </div>
                </div>
            `).join('');
        }

        // Initialize
        loadData();
    </script>
</body>
</html>
