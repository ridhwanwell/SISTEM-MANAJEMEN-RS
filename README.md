<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard - Hospital CMMS</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        .sidebar {
            width: 250px;
            height: 100vh;
            position: fixed;
            left: 0;
            top: 0;
            background: linear-gradient(180deg, #1e3a8a 0%, #1e40af 100%);
            color: white;
            overflow-y: auto;
            z-index: 1000;
        }
        .main-content {
            margin-left: 250px;
            padding: 2rem;
            background: #f8fafc;
            min-height: 100vh;
        }
        .stat-card {
            background: white;
            border-radius: 1rem;
            padding: 1.5rem;
            box-shadow: 0 4px 6px rgba(0,0,0,0.1);
            transition: transform 0.3s, box-shadow 0.3s;
        }
        .stat-card:hover {
            transform: translateY(-5px);
            box-shadow: 0 10px 20px rgba(0,0,0,0.15);
        }
        .nav-item {
            padding: 0.75rem 1.5rem;
            cursor: pointer;
            transition: all 0.3s;
            border-left: 4px solid transparent;
        }
        .nav-item:hover {
            background: rgba(255,255,255,0.1);
            border-left-color: white;
        }
        .nav-item.active {
            background: rgba(255,255,255,0.2);
            border-left-color: #fbbf24;
        }
        .badge {
            display: inline-block;
            padding: 0.25rem 0.75rem;
            border-radius: 9999px;
            font-size: 0.75rem;
            font-weight: 600;
        }
        .loading {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            display: none;
            align-items: center;
            justify-content: center;
            z-index: 9999;
        }
        .loading.active {
            display: flex;
        }
        .spinner {
            border: 4px solid #f3f3f3;
            border-top: 4px solid #3b82f6;
            border-radius: 50%;
            width: 50px;
            height: 50px;
            animation: spin 1s linear infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body>
    <!-- Loading -->
    <div id="loading" class="loading">
        <div class="spinner"></div>
    </div>

    <!-- Sidebar -->
    <div class="sidebar">
        <div class="p-6 border-b border-blue-700">
            <h1 class="text-2xl font-bold">🏥 Hospital CMMS</h1>
            <p class="text-blue-200 text-sm mt-1">Professional Edition</p>
        </div>

        <nav class="mt-4">
            <a href="dashboard.html" class="nav-item active flex items-center gap-3">
                <span class="text-xl">📊</span>
                <span>Dashboard</span>
            </a>
            <a href="admin.html" class="nav-item flex items-center gap-3">
                <span class="text-xl">🔧</span>
                <span>Equipment</span>
            </a>
            <a href="work-order.html" class="nav-item flex items-center gap-3">
                <span class="text-xl">📋</span>
                <span>Work Orders</span>
            </a>
            <a href="pm-schedule.html" class="nav-item flex items-center gap-3">
                <span class="text-xl">📅</span>
                <span>PM Schedules</span>
            </a>
            <a href="reports.html" class="nav-item flex items-center gap-3">
                <span class="text-xl">📈</span>
                <span>Reports</span>
            </a>
            <a href="user.html" class="nav-item flex items-center gap-3">
                <span class="text-xl">👤</span>
                <span>User Panel</span>
            </a>
        </nav>

        <div class="absolute bottom-0 w-full p-4 border-t border-blue-700">
            <div class="flex items-center gap-3 mb-3">
                <div class="w-10 h-10 rounded-full bg-blue-300 flex items-center justify-center font-bold text-blue-900">
                    <span id="userInitial">A</span>
                </div>
                <div class="flex-1">
                    <p class="text-sm font-semibold" id="userName">Admin</p>
                    <p class="text-xs text-blue-200" id="userRole">Administrator</p>
                </div>
            </div>
            <button onclick="logout()" class="w-full bg-red-500 hover:bg-red-600 text-white py-2 rounded-lg text-sm font-semibold">
                Logout
            </button>
        </div>
    </div>

    <!-- Main Content -->
    <div class="main-content">
        <!-- Header -->
        <div class="flex justify-between items-center mb-8">
            <div>
                <h1 class="text-3xl font-bold text-gray-800">Dashboard</h1>
                <p class="text-gray-600 mt-1">Welcome back! Here's your overview</p>
            </div>
            <div class="text-right">
                <p class="text-sm text-gray-600">Last Updated</p>
                <p class="text-lg font-semibold text-gray-800" id="lastUpdated">-</p>
            </div>
        </div>

        <!-- Stats Cards -->
        <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
            <!-- Total Equipment -->
            <div class="stat-card">
                <div class="flex items-center justify-between mb-4">
                    <div>
                        <p class="text-gray-600 text-sm">Total Equipment</p>
                        <h3 class="text-3xl font-bold text-gray-800" id="totalEquipment">0</h3>
                    </div>
                    <div class="w-16 h-16 rounded-full bg-blue-100 flex items-center justify-center">
                        <span class="text-3xl">🔧</span>
                    </div>
                </div>
                <div class="flex items-center gap-2 text-sm">
                    <span class="text-green-600 font-semibold">↑ 12%</span>
                    <span class="text-gray-600">vs last month</span>
                </div>
            </div>

            <!-- Active Work Orders -->
            <div class="stat-card">
                <div class="flex items-center justify-between mb-4">
                    <div>
                        <p class="text-gray-600 text-sm">Active Work Orders</p>
                        <h3 class="text-3xl font-bold text-gray-800" id="activeWorkOrders">0</h3>
                    </div>
                    <div class="w-16 h-16 rounded-full bg-orange-100 flex items-center justify-center">
                        <span class="text-3xl">📋</span>
                    </div>
                </div>
                <div class="flex items-center gap-2 text-sm">
                    <span class="text-orange-600 font-semibold" id="pendingWO">0 Pending</span>
                    <span class="text-gray-600">• </span>
                    <span class="text-blue-600 font-semibold" id="inProgressWO">0 In Progress</span>
                </div>
            </div>

            <!-- Maintenance This Month -->
            <div class="stat-card">
                <div class="flex items-center justify-between mb-4">
                    <div>
                        <p class="text-gray-600 text-sm">Maintenance (Month)</p>
                        <h3 class="text-3xl font-bold text-gray-800" id="maintenanceMonth">0</h3>
                    </div>
                    <div class="w-16 h-16 rounded-full bg-green-100 flex items-center justify-center">
                        <span class="text-3xl">✅</span>
                    </div>
                </div>
                <div class="flex items-center gap-2 text-sm">
                    <span class="text-green-600 font-semibold">On Track</span>
                    <span class="text-gray-600">maintenance schedule</span>
                </div>
            </div>

            <!-- Upcoming PM -->
            <div class="stat-card">
                <div class="flex items-center justify-between mb-4">
                    <div>
                        <p class="text-gray-600 text-sm">Upcoming PM (7d)</p>
                        <h3 class="text-3xl font-bold text-gray-800" id="upcomingPM">0</h3>
                    </div>
                    <div class="w-16 h-16 rounded-full bg-purple-100 flex items-center justify-center">
                        <span class="text-3xl">📅</span>
                    </div>
                </div>
                <div class="flex items-center gap-2 text-sm">
                    <span class="text-purple-600 font-semibold">Schedule Review</span>
                    <span class="text-gray-600">needed</span>
                </div>
            </div>
        </div>

        <!-- Charts Row -->
        <div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
            <!-- Equipment Status Chart -->
            <div class="bg-white rounded-xl shadow-lg p-6">
                <h3 class="text-xl font-bold text-gray-800 mb-4">Equipment by Status</h3>
                <canvas id="equipmentStatusChart"></canvas>
            </div>

            <!-- Work Orders Trend -->
            <div class="bg-white rounded-xl shadow-lg p-6">
                <h3 class="text-xl font-bold text-gray-800 mb-4">Work Orders by Priority</h3>
                <canvas id="workOrdersChart"></canvas>
            </div>
        </div>

        <!-- Recent Activity -->
        <div class="bg-white rounded-xl shadow-lg p-6">
            <h3 class="text-xl font-bold text-gray-800 mb-4">Recent Maintenance Activity</h3>
            <div id="recentActivity" class="space-y-3">
                <!-- Activity items will be inserted here -->
            </div>
        </div>
    </div>

    <script>
        const API_URL = 'http://localhost:3000/api';
        let token = localStorage.getItem('token');
        let currentUser = null;

        // Check authentication
        if (!token) {
            window.location.href = 'login.html';
        }

        // Parse JWT to get user info
        function parseJWT(token) {
            try {
                const base64Url = token.split('.')[1];
                const base64 = base64Url.replace(/-/g, '+').replace(/_/g, '/');
                const jsonPayload = decodeURIComponent(atob(base64).split('').map(c => {
                    return '%' + ('00' + c.charCodeAt(0).toString(16)).slice(-2);
                }).join(''));
                return JSON.parse(jsonPayload);
            } catch (e) {
                return null;
            }
        }

        currentUser = parseJWT(token);
        if (currentUser) {
            document.getElementById('userName').textContent = currentUser.username;
            document.getElementById('userRole').textContent = currentUser.role;
            document.getElementById('userInitial').textContent = currentUser.username.charAt(0).toUpperCase();
        }

        // Show/Hide loading
        function showLoading() {
            document.getElementById('loading').classList.add('active');
        }

        function hideLoading() {
            document.getElementById('loading').classList.remove('active');
        }

        // Fetch with auth
        async function fetchWithAuth(url, options = {}) {
            const defaultOptions = {
                headers: {
                    'Authorization': `Bearer ${token}`,
                    'Content-Type': 'application/json',
                    ...options.headers
                }
            };
            return fetch(url, { ...options, ...defaultOptions });
        }

        // Load dashboard data
        async function loadDashboard() {
            showLoading();
            try {
                const response = await fetchWithAuth(`${API_URL}/dashboard/stats`);
                if (!response.ok) throw new Error('Failed to load dashboard');
                
                const stats = await response.json();
                
                // Update stats cards
                document.getElementById('totalEquipment').textContent = stats.totalEquipment || 0;
                document.getElementById('maintenanceMonth').textContent = stats.maintenanceThisMonth || 0;
                document.getElementById('upcomingPM').textContent = stats.upcomingPM || 0;
                
                // Calculate work orders
                const woByStatus = stats.workOrdersByStatus || [];
                let totalActive = 0;
                let pending = 0;
                let inProgress = 0;
                
                woByStatus.forEach(wo => {
                    if (wo.status === 'pending') pending = wo.count;
                    if (wo.status === 'in_progress') inProgress = wo.count;
                    if (wo.status !== 'completed') totalActive += wo.count;
                });
                
                document.getElementById('activeWorkOrders').textContent = totalActive;
                document.getElementById('pendingWO').textContent = `${pending} Pending`;
                document.getElementById('inProgressWO').textContent = `${inProgress} In Progress`;
                
                // Update charts
                updateEquipmentChart(stats.equipmentByStatus || []);
                updateWorkOrdersChart(stats.workOrdersByStatus || []);
                
                // Load recent activity
                await loadRecentActivity();
                
                // Update timestamp
                document.getElementById('lastUpdated').textContent = new Date().toLocaleTimeString('id-ID');
                
            } catch (error) {
                console.error('Error loading dashboard:', error);
            } finally {
                hideLoading();
            }
        }

        // Update Equipment Status Chart
        function updateEquipmentChart(data) {
            const ctx = document.getElementById('equipmentStatusChart').getContext('2d');
            
            const labels = data.map(d => {
                if (d.status === 'operational') return 'Operational';
                if (d.status === 'maintenance') return 'Under Maintenance';
                if (d.status === 'broken') return 'Broken';
                return d.status;
            });
            
            const values = data.map(d => d.count);
            
            new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: values,
                        backgroundColor: [
                            '#10b981',
                            '#f59e0b',
                            '#ef4444'
                        ],
                        borderWidth: 2,
                        borderColor: '#fff'
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            position: 'bottom'
                        }
                    }
                }
            });
        }

        // Update Work Orders Chart
        function updateWorkOrdersChart(data) {
            const ctx = document.getElementById('workOrdersChart').getContext('2d');
            
            const labels = data.map(d => {
                if (d.status === 'pending') return 'Pending';
                if (d.status === 'in_progress') return 'In Progress';
                if (d.status === 'completed') return 'Completed';
                return d.status;
            });
            
            const values = data.map(d => d.count);
            
            new Chart(ctx, {
                type: 'bar',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'Work Orders',
                        data: values,
                        backgroundColor: [
                            '#f59e0b',
                            '#3b82f6',
                            '#10b981'
                        ],
                        borderRadius: 8
                    }]
                },
                options: {
                    responsive: true,
                    plugins: {
                        legend: {
                            display: false
                        }
                    },
                    scales: {
                        y: {
                            beginAtZero: true,
                            ticks: {
                                stepSize: 1
                            }
                        }
                    }
                }
            });
        }

        // Load recent activity
        async function loadRecentActivity() {
            try {
                const response = await fetchWithAuth(`${API_URL}/reports/maintenance?limit=5`);
                if (!response.ok) throw new Error('Failed to load activity');
                
                const records = await response.json();
                const container = document.getElementById('recentActivity');
                
                if (records.length === 0) {
                    container.innerHTML = '<p class="text-gray-500 text-center py-4">No recent activity</p>';
                    return;
                }
                
                container.innerHTML = records.slice(0, 5).map(record => {
                    const data = JSON.parse(record.data);
                    const date = new Date(record.timestamp).toLocaleString('id-ID');
                    
                    let icon = '📋';
                    let bgColor = 'bg-blue-100';
                    let textColor = 'text-blue-800';
                    
                    if (record.type === 'kalibrasi') {
                        icon = '✅';
                        bgColor = 'bg-green-100';
                        textColor = 'text-green-800';
                    } else if (record.type === 'perbaikan_pemeliharaan') {
                        icon = '🔧';
                        bgColor = 'bg-orange-100';
                        textColor = 'text-orange-800';
                    }
                    
                    return `
                        <div class="flex items-center gap-4 p-4 bg-gray-50 rounded-lg hover:bg-gray-100 transition">
                            <div class="w-12 h-12 rounded-full ${bgColor} flex items-center justify-center text-2xl">
                                ${icon}
                            </div>
                            <div class="flex-1">
                                <p class="font-semibold text-gray-800">${record.namaAlat}</p>
                                <p class="text-sm text-gray-600">SN: ${record.snAlat} • ${date}</p>
                                <p class="text-sm ${textColor} mt-1">${record.performedBy || 'Unknown'}</p>
                            </div>
                            <span class="px-3 py-1 rounded-full text-xs font-semibold ${bgColor} ${textColor}">
                                ${record.type === 'kalibrasi' ? 'Kalibrasi' : 'Perbaikan'}
                            </span>
                        </div>
                    `;
                }).join('');
                
            } catch (error) {
                console.error('Error loading activity:', error);
            }
        }

        // Logout
        function logout() {
            if (confirm('Are you sure you want to logout?')) {
                localStorage.removeItem('token');
                window.location.href = 'login.html';
            }
        }

        // Auto refresh every 5 minutes
        setInterval(loadDashboard, 300000);

        // Initialize
        loadDashboard();
    </script>
</body>
</html>
