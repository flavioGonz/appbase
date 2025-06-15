# appbase




<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>inventariOS - Gestión de Inventario Georreferenciado</title>
    <!-- Dependencias CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css">
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

    <style>
        :root {
            /* Tema Oscuro (Default) */
            --bg-color: #2c3e50; --mid-bg-color: #34495e; --light-bg-color: #3f5166;
            --text-color: #ecf0f1; --subtle-text-color: #bdc3c7; --border-color: rgba(255, 255, 255, 0.1);
            --primary-color: #1abc9c; --primary-dark-color: #16a085;
            --glass-bg: rgba(44, 62, 80, 0.8); --glass-blur: 18px;
            --input-bg: rgba(0, 0, 0, 0.3); --input-border: rgba(255, 255, 255, 0.2);
            --box-shadow-medium: 0 8px 25px rgba(0,0,0,.45);
            /* Colores de Estado */
            --status-ok: #2ecc71; --status-warn: #f39c12; --status-danger: #e74c3c;
            --status-info: #3498db; --status-special: #9b59b6;
        }
        body.theme-light {
            /* Tema Claro */
            --bg-color: #f8f9fa; --mid-bg-color: #e9ecef; --light-bg-color: #ffffff;
            --text-color: #212529; --subtle-text-color: #6c757d; --border-color: #dee2e6;
            --primary-color: #007bff; --primary-dark-color: #0056b3;
            --glass-bg: rgba(255, 255, 255, 0.85); --glass-blur: 10px;
            --input-bg: #ffffff; --input-border: #ced4da;
            --box-shadow-medium: 0 8px 25px rgba(0,0,0,.12);
        }
        /* Estilos Globales y Animaciones */
        body { font-family: 'Poppins', sans-serif; background: var(--bg-color); color: var(--text-color); transition: background-color 0.4s ease, color 0.4s ease; overflow: hidden; }
        .background-overlay { position: fixed; top: 0; left: 0; width: 100vw; height: 100vh; background: linear-gradient(135deg, var(--bg-color) 0%, var(--mid-bg-color) 100%); z-index: -1; }
        .content-wrapper, .installer-content-wrapper { position: absolute; top: 0; left: 0; width: 100%; height: 100vh; padding-top: 65px; box-sizing: border-box; overflow-y: auto; z-index: 10; 
            scrollbar-width: thin; scrollbar-color: var(--primary-color) var(--mid-bg-color);
        }
        .content-wrapper::-webkit-scrollbar, .installer-content-wrapper::-webkit-scrollbar { width: 8px; }
        .content-wrapper::-webkit-scrollbar-track, .installer-content-wrapper::-webkit-scrollbar-track { background: var(--mid-bg-color); }
        .content-wrapper::-webkit-scrollbar-thumb, .installer-content-wrapper::-webkit-scrollbar-thumb { background-color: var(--primary-color); border-radius: 6px; }

        .view-section { background-color: var(--glass-bg); backdrop-filter: blur(var(--glass-blur)); padding: 30px; border-radius: 15px; box-shadow: var(--box-shadow-medium); margin: 20px auto; width: 95%; max-width: 1600px; border: 1px solid var(--border-color); opacity: 0; transform: translateY(20px); transition: opacity 0.5s ease-out, transform 0.5s ease-out; display: none; }
        .view-section.active { display: block; opacity: 1; transform: translateY(0); }
        #geolocation-view { padding: 0; height: calc(100vh - 105px); overflow: hidden; }
        #geolocation-map, .modal-map { width: 100%; height: 100%; border-radius: 15px; }
        /* Componentes UI */
        .navbar { position: sticky; top: 0; width: 100%; background-color: var(--glass-bg); backdrop-filter: blur(var(--glass-blur)); box-shadow: 0 4px 12px rgba(0,0,0,.3); z-index: 1000; border-bottom: 1px solid var(--border-color); }
        .section-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 2rem; border-bottom: 1px solid var(--border-color); padding-bottom: 1rem; }
        .section-header h2 { margin: 0; color: var(--primary-color); font-weight: 700; font-size: 2rem; }
        .table-hover>tbody>tr:hover { background-color: rgba(26, 188, 156, 0.15); cursor: pointer; }
        body.theme-light .table-hover>tbody>tr:hover { background-color: rgba(0, 123, 255, 0.1); }
        .item-thumbnail { width: 45px; height: 45px; object-fit: cover; border-radius: 8px; margin-right: 15px; border: 2px solid var(--border-color); transition: transform .2s ease; }
        .table-hover>tbody>tr:hover .item-thumbnail { transform: scale(2.5) translateX(20px); z-index: 10; }
        .modal.fade .modal-dialog { transition: transform .4s ease-out; transform: scale(0.9) translateY(-20px); }
        .modal.show .modal-dialog { transform: scale(1) translateY(0); }
        .modal-content { border-radius: 15px; box-shadow: var(--box-shadow-medium); border: none; background-color: var(--glass-bg); color: var(--text-color); backdrop-filter: blur(var(--glass-blur)); }
        .modal-header { color: var(--primary-color); border-bottom-color: var(--border-color); }
        .modal-body .form-label { color: var(--primary-color); font-weight: 500; }
        .modal-body .form-control, .modal-body .form-select { background-color: var(--input-bg); border: 1px solid var(--input-border); color: var(--text-color); border-radius: 8px; }
        .nav-tabs { border-bottom-color: var(--border-color); }
        .nav-tabs .nav-link { color: var(--subtle-text-color); border-color: transparent; }
        .nav-tabs .nav-link.active { color: var(--primary-color); background-color: rgba(0,0,0,0.2); border-color: var(--border-color) var(--border-color) transparent; }
        body.theme-light .nav-tabs .nav-link.active { background-color: var(--mid-bg-color); }
        .tab-content { background-color: rgba(0,0,0,0.1); padding: 1.5rem; border: 1px solid var(--border-color); border-top: none; border-radius: 0 0 .5rem .5rem; }
        body.theme-light .tab-content { background-color: transparent; }
        .toast-container { z-index: 2050; }
        .toast { background-color: var(--light-bg-color); color: var(--text-color); border: 1px solid var(--border-color); }
        /* Badges de Estado */
        .badge { font-weight: 500; font-size: 0.8rem; padding: .4em .7em; }
        .badge-status-activo, .badge-status-finalizada, .badge-status-aprobada { background-color: var(--status-ok); color: #1a1a1a; }
        .badge-status-almacén, .badge-status-arrendado, .badge-status-venciendo { background-color: var(--status-warn); color: #1a1a1a; }
        .badge-status-reparación, .badge-status-inactivo { background-color: var(--status-danger); color: var(--text-color); }
        .badge-status-baja, .badge-status-vendido { background-color: #95a5a6; color: var(--text-color); }
        .badge-status-pendiente, .badge-status-en-progreso, .badge-status-asignada, .badge-status-en-revisión { background-color: var(--status-info); color: var(--text-color); }
        .badge-status-relevamiento { background-color: var(--status-special); color: var(--text-color); }
        .warranty-expiring { color: var(--status-warn); font-weight: bold; }
        .warranty-expired { color: var(--status-danger); font-weight: bold; }
        .warranty-active { color: var(--status-ok); }
        /* Dashboard & Inventory Cards */
        .kpi-card { background: var(--light-bg-color); border-radius: 15px; padding: 20px; text-align: center; transition: all 0.3s ease; border: 1px solid var(--border-color); cursor: pointer; }
        .kpi-card:hover { transform: translateY(-10px); box-shadow: var(--box-shadow-medium); border-color: var(--primary-color); }
        .kpi-card .icon { font-size: 2.5rem; margin-bottom: 10px; color: var(--primary-color); }
        .kpi-card .value { font-size: 2.5rem; font-weight: 700; }
        .kpi-card .label { font-size: 1rem; color: var(--subtle-text-color); }
        /* Wizard Modal */
        .wizard-step { display: none; }
        .wizard-step.active { display: block; animation: fadeIn 0.5s; }
        @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
        .progress-bar { transition: width .6s ease; }
        /* Estilos del Modal de Inventario */
        .product-image-carousel .carousel-inner { border-radius: .5rem; background-color: rgba(0,0,0,0.2); }
        .product-image-carousel .carousel-item img { height: 300px; object-fit: contain; }
        .code-container img { max-width: 100%; height: auto; background: white; padding: 10px; border-radius: .5rem; }
        /* Otros estilos... */
        #login-screen { display: flex; height: 100vh; width: 100vw; position: fixed; top: 0; left: 0; z-index: 2000; background-color: var(--bg-color); transition: opacity 0.8s ease-in-out, visibility 0.8s; opacity: 1; visibility: visible; }
        #login-screen.hidden { opacity: 0; visibility: hidden; pointer-events: none; }
        /* --- MODIFICADO --- Se usa la imagen local */
        .login-image-half { flex: 1; background-image: linear-gradient(to right, rgba(0, 0, 0, 0.4), var(--bg-color)), url('images/login-image.webp'); background-size: cover; background-position: center; animation: fadeInImage 1.2s ease-out; }
        @keyframes fadeInImage { from { opacity: 0; transform: scale(1.1); } to { opacity: 1; transform: scale(1); } }
        .login-form-half { flex: 1; display: flex; justify-content: center; align-items: center; padding: 2rem; background-color: var(--bg-color); animation: slideInForm 1s ease-out forwards; opacity: 0; }
        @keyframes slideInForm { from { transform: translateX(50px); opacity: 0; } to { transform: translateX(0); opacity: 1; } }
        .login-form-container { width: 80%; max-width: 450px; background: var(--mid-bg-color); padding: 3rem; border-radius: 12px; box-shadow: var(--box-shadow-medium); text-align: center; }
        
        .task-card { background: var(--light-bg-color); border: 1px solid var(--border-color); border-left: 5px solid var(--primary-color); border-radius: 8px; padding: 15px; transition: all .2s ease; cursor: pointer;}
        .task-card:hover { transform: translateY(-5px); box-shadow: var(--box-shadow-medium); }
        #installation-photos-preview { display: flex; flex-wrap: wrap; gap: 10px; margin-top: 15px; }
        .photo-thumbnail { position: relative; width: 100px; height: 100px; }
        .photo-thumbnail img { width: 100%; height: 100%; object-fit: cover; border-radius: 8px; }
        .photo-thumbnail .remove-photo { position: absolute; top: -5px; right: -5px; background: #e74c3c; color: white; border-radius: 50%; width: 20px; height: 20px; border: none; display: flex; align-items: center; justify-content: center; font-size: 12px; cursor: pointer; }
        .material-request-box { border: 2px dashed var(--status-warn); padding: 1rem; border-radius: .5rem; background-color: rgba(243, 156, 18, 0.1); }
    </style>
</head>
<body class="theme-dark">

    <div class="background-overlay"></div>
    <div class="toast-container position-fixed top-0 end-0 p-3"></div>
    
    <div id="login-screen">
        <div class="login-image-half d-none d-md-block"></div>
        <div class="login-form-half">
            <div class="login-form-container">
                <h2>Bienvenido a inventariOS</h2>
                <form id="loginForm">
                    <div class="mb-3"><input type="text" class="form-control form-control-lg" id="loginUsername" placeholder="Usuario (admin/empresa/installer)" required value="installer"></div>
                    <div class="mb-4"><input type="password" class="form-control form-control-lg" id="loginPassword" placeholder="Contraseña (admin/empresa/installer)" required value="installer"></div>
                    <button type="submit" class="btn btn-primary btn-lg w-100 mb-3">Ingresar</button>
                </form>
            </div>
        </div>
    </div>

    <div id="main-app" style="display: none;">
        <nav class="navbar navbar-expand-lg">
            <div class="container-fluid">
                <a class="navbar-brand" href="#"><i class="fas fa-cubes me-2"></i>inventariOS</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"><span class="navbar-toggler-icon"></span></button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav me-auto" id="main-nav-links">
                        <!-- Links se generarán dinámicamente -->
                    </ul>
                    <ul class="navbar-nav ms-auto d-flex align-items-center">
                        <li class="nav-item">
                            <button id="themeToggleBtn" title="Cambiar Tema" class="btn btn-outline-secondary border-0 me-2"><i class="fas fa-sun"></i></button>
                        </li>
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown"><i class="fas fa-user-circle me-1"></i>Usuario</a>
                            <ul class="dropdown-menu dropdown-menu-end">
                                <li><a class="dropdown-item" href="#" id="logoutBtnNav"><i class="fas fa-sign-out-alt me-2"></i>Cerrar Sesión</a></li>
                            </ul>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
        <div class="content-wrapper">
            <!-- Vistas de cada sección -->
            <div id="dashboard-view" class="view-section">
                <div class="section-header"><h2><i class="fas fa-tachometer-alt me-2"></i>Dashboard</h2></div>
                <div id="kpi-cards-container" class="row g-4"></div>
            </div>
            <div id="geolocation-view" class="view-section"><div id="geolocation-map"></div></div>
            <div id="companies-view" class="view-section">
                <div class="section-header"><h2><i class="fas fa-building me-2"></i>Empresas</h2><button class="btn btn-primary" id="btn-new-company"><i class="fas fa-plus me-2"></i>Nueva Empresa</button></div>
                <div class="table-responsive"><table class="table table-hover"><thead><tr><th>ID</th><th>Nombre</th><th>Contacto</th><th>Email</th></tr></thead><tbody id="companiesTableBody"></tbody></table></div>
            </div>
            <div id="clients-view" class="view-section">
                <div class="section-header"><h2><i class="fas fa-users me-2"></i>Clientes</h2><button class="btn btn-primary" id="btn-new-client"><i class="fas fa-plus me-2"></i>Nuevo Cliente</button></div>
                <div class="table-responsive"><table class="table table-hover"><thead><tr><th>ID</th><th>Nombre</th><th>Empresa</th><th>Contrato</th><th>Estado</th></tr></thead><tbody id="clientsTableBody"></tbody></table></div>
            </div>
            <div id="inventory-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-boxes-stacked me-2"></i>Inventario</h2>
                    <button class="btn btn-primary" id="btn-new-inventory"><i class="fas fa-plus me-2"></i>Nuevo Artículo</button>
                </div>
                <!-- KPIs y Gráfico de Inventario -->
                <div class="row mb-4 g-4">
                    <div class="col-lg-8">
                        <div id="inventory-kpi-cards" class="row g-4"></div>
                    </div>
                    <div class="col-lg-4">
                        <div class="kpi-card h-100">
                            <canvas id="inventoryChart"></canvas>
                        </div>
                    </div>
                </div>
                <!-- Filtros y Tabla -->
                <div class="d-flex flex-wrap gap-2 mb-3">
                    <input type="text" id="inventory-search" class="form-control form-control-sm w-auto" placeholder="Buscar por serial, descripción...">
                </div>
                <div class="table-responsive">
                    <table class="table table-hover align-middle">
                        <thead><tr><th></th><th>Serial/ID</th><th>Descripción</th><th>Estado</th><th>Ubicación</th><th>Garantía</th></tr></thead>
                        <tbody id="inventoryTableBody"></tbody>
                    </table>
                </div>
            </div>
            <div id="users-view" class="view-section"><div class="section-header"><h2><i class="fas fa-user-shield me-2"></i>Usuarios</h2></div><div class="table-responsive"><table class="table table-hover"><thead><tr><th>ID</th><th>Usuario</th><th>Nombre</th><th>Rol</th><th>Empresa</th></tr></thead><tbody id="usersTableBody"></tbody></table></div></div>
            <div id="providers-view" class="view-section"><div class="section-header"><h2><i class="fas fa-truck me-2"></i>Proveedores</h2></div><div class="table-responsive"><table class="table table-hover"><thead><tr><th>ID</th><th>Nombre</th><th>Contacto</th></tr></thead><tbody id="providersTableBody"></tbody></table></div></div>
            <div id="installations-view" class="view-section">
                <div class="section-header"><h2><i class="fas fa-tools me-2"></i>Instalaciones</h2><button class="btn btn-primary" id="btn-new-installation"><i class="fas fa-plus me-2"></i>Nueva Tarea</button></div>
                <div class="table-responsive"><table class="table table-hover"><thead><tr><th>ID</th><th>Cliente</th><th>Fecha</th><th>Responsable</th><th>Estado</th><th></th></tr></thead><tbody id="installationsTableBody"></tbody></table></div>
            </div>
            <div id="services-view" class="view-section"><div class="section-header"><h2><i class="fas fa-headset me-2"></i>Servicios</h2></div><div class="table-responsive"><table class="table table-hover"><thead><tr><th>ID</th><th>Cliente</th><th>Tipo</th><th>Estado</th></tr></thead><tbody id="servicesTableBody"></tbody></table></div></div>
            <div id="contracts-view" class="view-section">
                <div class="section-header"><h2><i class="fas fa-file-signature me-2"></i>Contratos</h2></div>
                <div class="table-responsive"><table class="table table-hover"><thead><tr><th>Cliente</th><th>Tipo Contrato</th><th>Fecha Fin</th><th>Estado</th></tr></thead><tbody id="contractsTableBody"></tbody></table></div>
            </div>
            <div id="subcontracts-view" class="view-section"><div class="section-header"><h2><i class="fas fa-handshake me-2"></i>Sub Contratos</h2></div>
                <div class="table-responsive"><table class="table table-hover"><thead><tr><th></th><th>Contratista</th><th>Asignadas</th><th>Terminadas</th><th>Estado</th></tr></thead><tbody id="subcontractsTableBody"></tbody></table></div>
            </div>
        </div>
    </div>

    <!-- --- MODIFICADO --- Contenedor del instalador ahora usa una tabla -->
    <div id="installer-app-container" style="display: none;">
        <nav class="navbar installer-top-bar">
            <div class="container-fluid">
                <a class="navbar-brand" href="#"><i class="fas fa-hard-hat me-2"></i>Portal Instalador</a>
                <ul class="navbar-nav ms-auto d-flex align-items-center flex-row">
                     <li class="nav-item">
                        <button id="installerThemeToggleBtn" title="Cambiar Tema" class="btn btn-outline-secondary border-0 me-2"><i class="fas fa-sun"></i></button>
                    </li>
                    <li class="nav-item dropdown">
                        <a class="nav-link dropdown-toggle" href="#" id="installerNavbarDropdown" role="button" data-bs-toggle="dropdown"><i class="fas fa-user-circle me-1"></i>Instalador</a>
                        <ul class="dropdown-menu dropdown-menu-end"><li><a class="dropdown-item" href="#" id="logoutInstallerBtnNav"><i class="fas fa-sign-out-alt me-2"></i>Cerrar Sesión</a></li></ul>
                    </li>
                </ul>
            </div>
        </nav>
        <div class="installer-content-wrapper">
            <div id="installer-dashboard-view" class="view-section active">
                <div class="section-header"><h2><i class="fas fa-tasks me-2"></i>Mis Tareas Asignadas</h2></div>
                <div class="table-responsive">
                    <table class="table table-hover align-middle">
                        <thead>
                            <tr>
                                <th>Estado</th>
                                <th>Cliente</th>
                                <th>Fecha Solicitud</th>
                                <th>Fecha Finalizado</th>
                                <th>Acciones</th>
                            </tr>
                        </thead>
                        <tbody id="installer-tasks-table-body"></tbody>
                    </table>
                </div>
            </div>
        </div>
    </div>
    
    <div id="modals-container"></div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

    <script>
    (function() {
        'use strict';

        // --- DATA SIMULATION ---
        let companies = [ { id: 1, name: "SecureTech Uruguay", nif: "B12345678", address: "Av. 18 de Julio 1234, Montevideo", phone: "29001122", email: "contacto@securetech.uy", contact: "Juan Pérez", lat: -34.9063, lon: -56.1891 }, { id: 2, name: "Conectividad Total S.A.", nif: "A87654321", address: "Bv. Artigas 888, Montevideo", phone: "24005566", email: "info@conectividadtotal.com", contact: "Maria García", lat: -34.8933, lon: -56.1667 } ];
        let clients = [ { id: 101, fullName: "Edificio Central Park", dni: "211234560018", companyId: 1, address: "Av. Brasil 2525, Montevideo", phone: "099111222", email: "admin@centralpark.uy", notes: "Cliente residencial premium.", lat: -34.9113, lon: -56.1535, status: "Activo" }, { id: 102, fullName: "Oficinas World Trade Center", dni: "218765430019", companyId: 1, address: "Av. Dr. Luis Alberto de Herrera 1248, Montevideo", phone: "26261234", email: "seguridad@wtc.com.uy", notes: "Requiere equipos de alta gama.", lat: -34.8958, lon: -56.1428, status: "Activo" }, { id: 103, fullName: "Farmacia Pocitos", dni: "120987650011", companyId: 2, address: "Benito Blanco 980, Montevideo", phone: "098333444", email: "farmacia.pocitos@gmail.com", notes: "", lat: -34.9183, lon: -56.1501, status: "Inactivo" } ];
        let inventoryItems = [ { id: 1, serial: "CAM-001", label: "Cámara Domo 4MP", type: "Cámara IP", brand: "Hikvision", model: "DS-2CD2143G0-I", status: "Instalado", location: "Cliente", clientId: 101, installationDate: "2023-01-15", warrantyPeriodMonths: 24, thumbnail: "https://source.unsplash.com/random/150x150/?cctv,camera", history: [] }, { id: 2, serial: "DVR-001", label: "DVR 16 Canales AcuSense", type: "DVR", brand: "Hikvision", model: "iDS-7216HQHI-M2/S", status: "Instalado", location: "Cliente", clientId: 101, installationDate: "2023-01-15", warrantyPeriodMonths: 24, thumbnail: "https://source.unsplash.com/random/150x150/?server,dvr", history: [] }, { id: 3, serial: "SW-001", label: "Switch PoE 8 Puertos", type: "Switch", brand: "TP-Link", model: "TL-SG108PE", status: "Almacén", location: "Almacén Central", clientId: null, installationDate: null, warrantyPeriodMonths: 12, thumbnail: "https://source.unsplash.com/random/150x150/?network,switch", history: [] }, { id: 4, serial: "ALM-001", label: "Panel de Alarma AX Pro", type: "Alarma", brand: "Hikvision", model: "DS-PWA96-M-WE", status: "Almacén", location: "Almacén Central", clientId: null, installationDate: null, warrantyPeriodMonths: 36, thumbnail: "https://source.unsplash.com/random/150x150/?alarm,security", history: [] }, { id: 5, serial: "CAM-002", label: "Cámara Bullet Varifocal", type: "Cámara IP", brand: "Dahua", model: "IPC-HFW5442E-ZE", status: "Arrendado", location: "Cliente", clientId: 102, installationDate: "2024-06-20", warrantyPeriodMonths: 1, thumbnail: "https://source.unsplash.com/random/150x150/?security,camera", history: [] }, { id: 6, serial: "NVR-001", label: "NVR 32 Canales 4K", type: "NVR", brand: "Dahua", model: "NVR5232-4KS2", status: "Reparación", location: "Taller", clientId: 102, installationDate: "2022-05-10", warrantyPeriodMonths: 24, thumbnail: "https://source.unsplash.com/random/150x150/?electronics,repair", history: [] } ];
        let users = [ { id: 1, username: "admin", password: "admin", role: "Superadmin", fullName: "Admin General", companyId: null }, { id: 2, username: "empresa", password: "empresa", role: "Empresa", fullName: "Gerente SecureTech", companyId: 1 }, { id: 201, username: "installer", password: "installer", role: "Instalador", fullName: "Pedro García", companyId: 1 } ];
        // --- MODIFICADO --- Se añaden nuevos campos a las instalaciones
        let installations = [ { id: 1, clientId: 101, companyId: 1, installerId: 201, date: "2023-01-15", description: "Instalación de sistema CCTV completo en Edificio Central Park.", status: "Aprobada", equipmentSerials: ["CAM-001", "DVR-001"], requiredEquipment: ["CAM-001", "DVR-001"], photoUrls:[], notes: "Cliente conforme. Sistema funcionando 100%.", completionDate: "2023-01-16", materialRequest: null }, { id: 2, clientId: 102, companyId: 1, installerId: 201, date: new Date().toISOString().split('T')[0], description: "Instalación de cámaras perimetrales en WTC.", status: "Asignada", equipmentSerials: [], requiredEquipment: ["CAM-002", "NVR-001"], photoUrls:[], notes: "Iniciando cableado en torre 1.", completionDate: null, materialRequest: null }, { id: 3, clientId: 103, companyId: 2, installerId: null, date: new Date().toISOString().split('T')[0], description: "Relevamiento para sistema de alarmas.", status: "Pendiente", equipmentSerials: [], requiredEquipment: ["ALM-001"], photoUrls:[], notes: "", completionDate: null, materialRequest: null } ];
        let contracts = [ { id: 1, type: "Mantenimiento", clientId: 101, startDate: "2023-01-01", endDate: "2025-01-01", status: "Activo" }, { id: 2, type: "Arriendo", clientId: 102, startDate: "2022-06-20", endDate: "2024-08-30", status: "Venciendo" } ];
        let subcontracts = [ { id: 1, contractor: "Instaladores Express", mainContractId: 1, status: "Activo", contactPhoto: "https://randomuser.me/api/portraits/men/32.jpg", subcontractorId: 901 } ];
        let providers = [{ id: 1, name: "DistriEquipos S.A.", contact: "Gerardo Lópes" }];
        let services = [{ id: 1, clientId: 101, type: "Mantenimiento", status: "Finalizada" }];

        // --- CORE APP STATE & HELPERS ---
        let currentUser = null;
        let currentView = 'dashboard';
        let inventoryChartInstance = null;
        let nextIdCounters = {};

        const getEl = (id) => document.getElementById(id);
        const qS = (selector) => document.querySelector(selector);
        const qSA = (selector) => document.querySelectorAll(selector);

        const generateId = (arr) => {
            const arrName = arr === companies ? 'companies' : arr === clients ? 'clients' : 'default';
            if (!nextIdCounters[arrName]) {
                nextIdCounters[arrName] = arr.length > 0 ? Math.max(...arr.map(item => item.id)) + 1 : 1;
            }
            return nextIdCounters[arrName]++;
        };

        const addHistoryEvent = (serial, type, description, responsible) => {
            const item = inventoryItems.find(i => i.serial === serial);
            if (item) {
                if (!item.history) item.history = [];
                item.history.unshift({ date: new Date().toISOString().split('T')[0], type, description, responsible: responsible || currentUser?.fullName || 'Sistema' });
            }
        };
        const showToast = (message, type = 'success') => {
            const toastId = `toast-${Date.now()}`;
            const toastIcon = type === 'success' ? 'fa-check-circle text-success' : 'fa-times-circle text-danger';
            const toastHTML = `<div id="${toastId}" class="toast" role="alert" aria-live="assertive" aria-atomic="true"><div class="toast-header"><i class="fas ${toastIcon} me-2"></i><strong class="me-auto">Notificación</strong><button type="button" class="btn-close btn-close-white" data-bs-dismiss="toast"></button></div><div class="toast-body">${message}</div></div>`;
            getEl('toast-container').insertAdjacentHTML('beforeend', toastHTML);
            new bootstrap.Toast(getEl(toastId)).show();
        };

        // --- THEME MANAGEMENT ---
        const applyTheme = (theme) => {
            document.body.className = `theme-${theme}`;
            const icon = theme === 'light' ? 'fa-moon' : 'fa-sun';
            qSA('#themeToggleBtn, #installerThemeToggleBtn').forEach(btn => btn.innerHTML = `<i class="fas ${icon}"></i>`);
            localStorage.setItem('inventariOS-theme', theme);
            if (currentView === 'geolocation' && getEl('geolocation-map')) renderGeolocationMap();
            if (currentUser?.role === 'Instalador') renderInstallerDashboard(currentUser);
        };
        [getEl('themeToggleBtn'), getEl('installerThemeToggleBtn')].forEach(btn => btn.addEventListener('click', () => applyTheme(document.body.classList.contains('theme-light') ? 'dark' : 'light')));
        
        // --- MAP INITIALIZATION ---
        const initMap = (mapId, center, zoom) => {
            const container = getEl(mapId);
            if (!container) return null;
            if (container._leaflet_id) { container._leaflet_map.remove(); }
            const isLight = document.body.classList.contains('theme-light');
            const tileUrl = isLight ? 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png' : 'https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png';
            const attribution = '© OpenStreetMap & Stadia Maps';
            const map = L.map(mapId, { center, zoom });
            L.tileLayer(tileUrl, { attribution, maxZoom: 19 }).addTo(map);
            container._leaflet_map = map;
            return map;
        };

        // --- VIEW & NAVIGATION ---
        const navConfig = {
            Superadmin: [
                { view: 'dashboard', icon: 'fa-tachometer-alt', label: 'Dashboard' },
                { label: 'Gestión', icon: 'fa-sitemap', submenu: [ { view: 'companies', icon: 'fa-building', label: 'Empresas' }, { view: 'clients', icon: 'fa-users', label: 'Clientes' }, { view: 'users', icon: 'fa-user-shield', label: 'Usuarios' }, { view: 'providers', icon: 'fa-truck', label: 'Proveedores' }, { view: 'geolocation', icon: 'fa-map-marked-alt', label: 'Geolocalización' } ] },
                { label: 'Operaciones', icon: 'fa-cogs', submenu: [ { view: 'inventory', icon: 'fa-boxes-stacked', label: 'Inventario' }, { view: 'installations', icon: 'fa-tools', label: 'Instalaciones' }, { view: 'services', icon: 'fa-headset', label: 'Servicios' } ] },
                { label: 'Finanzas', icon: 'fa-dollar-sign', submenu: [ { view: 'contracts', icon: 'fa-file-signature', label: 'Contratos' }, { view: 'subcontracts', icon: 'fa-handshake', label: 'Sub Contratos' } ] }
            ],
            // --- MODIFICADO --- Se añade acceso a proveedores para la empresa
            Empresa: [
                { view: 'dashboard', icon: 'fa-tachometer-alt', label: 'Dashboard' },
                { view: 'clients', icon: 'fa-users', label: 'Mis Clientes' },
                { view: 'inventory', icon: 'fa-boxes-stacked', label: 'Mi Inventario' },
                { view: 'installations', icon: 'fa-tools', label: 'Mis Instalaciones' },
                { view: 'providers', icon: 'fa-truck', label: 'Proveedores' }
            ]
        };

        function buildNavbar(role) {
            const navItems = navConfig[role];
            if (!navItems) return;
            getEl('main-nav-links').innerHTML = navItems.map(item => {
                if (item.submenu) {
                    return `<li class="nav-item dropdown"><a class="nav-link dropdown-toggle" href="#" role="button" data-bs-toggle="dropdown"><i class="fas ${item.icon} fa-fw me-2"></i>${item.label}</a><ul class="dropdown-menu">${item.submenu.map(sub => `<li><a class="dropdown-item" href="#" data-view="${sub.view}"><i class="fas ${sub.icon} fa-fw me-2"></i>${sub.label}</a></li>`).join('')}</ul></li>`;
                }
                return `<li class="nav-item"><a class="nav-link" href="#" data-view="${item.view}"><i class="fas ${item.icon} fa-fw me-2"></i>${item.label}</a></li>`;
            }).join('');
            qSA('#main-nav-links a[data-view]').forEach(link => link.addEventListener('click', (e) => { e.preventDefault(); displayView(e.currentTarget.dataset.view); }));
        }
        
        const displayView = (viewId) => {
            if (currentView === viewId && getEl(`${viewId}-view`).classList.contains('active')) return;
            currentView = viewId;
            qSA('.view-section').forEach(s => s.classList.remove('active'));
            qSA('#main-nav-links .nav-link, #main-nav-links .dropdown-item').forEach(l => l.classList.remove('active'));
            
            const targetView = getEl(`${viewId}-view`);
            if (targetView) {
                targetView.classList.add('active');
                setTimeout(() => targetView.style.opacity = 1, 10);
                const activeLink = qS(`#main-nav-links a[data-view="${viewId}"]`);
                if (activeLink) {
                    activeLink.classList.add('active');
                    const dropdown = activeLink.closest('.dropdown');
                    if(dropdown) dropdown.querySelector('.dropdown-toggle').classList.add('active');
                }
            }
            const renderFunctions = {
                dashboard: renderDashboard, geolocation: renderGeolocationMap, companies: renderCompaniesTable, clients: renderClientsTable,
                inventory: renderInventoryView, users: renderUsersTable, providers: renderProvidersTable,
                installations: renderInstallationsView, services: renderServicesTable,
                contracts: renderContractsTable, subcontracts: renderSubcontractsTable,
            };
            if (renderFunctions[viewId]) renderFunctions[viewId]();
        };

        // --- AUTHENTICATION ---
        getEl('loginForm').addEventListener('submit', (e) => {
            e.preventDefault();
            const user = users.find(u => u.username === getEl('loginUsername').value && u.password === getEl('loginPassword').value);
            if (user) {
                currentUser = user;
                getEl('login-screen').classList.add('hidden');
                if (user.role === 'Superadmin' || user.role === 'Empresa') {
                    getEl('main-app').style.display = 'block';
                    getEl('installer-app-container').style.display = 'none';
                    buildNavbar(user.role);
                    qS('#navbarDropdown').innerHTML = `<i class="fas fa-user-circle me-1"></i> ${user.fullName}`;
                    displayView('dashboard');
                } else if (user.role === 'Instalador') {
                    getEl('main-app').style.display = 'none';
                    getEl('installer-app-container').style.display = 'block';
                    qS('#installerNavbarDropdown').innerHTML = `<i class="fas fa-user-circle me-1"></i> ${user.fullName}`;
                    renderInstallerDashboard(user);
                }
            } else { showToast('Credenciales incorrectas.', 'danger'); }
        });
        const logout = () => window.location.reload();
        getEl('logoutBtnNav').addEventListener('click', logout);
        getEl('logoutInstallerBtnNav').addEventListener('click', logout);

        // --- DASHBOARD ---
        function renderDashboard() {
            const isSuperAdmin = currentUser.role === 'Superadmin';
            const companyId = currentUser.companyId;
            const clientData = isSuperAdmin ? clients : clients.filter(c => c.companyId === companyId);
            const inventoryData = isSuperAdmin ? inventoryItems : inventoryItems.filter(i => { const client = clients.find(c => c.id === i.clientId); return (client && client.companyId === companyId) || i.status === 'Almacén'; });
            const installationData = isSuperAdmin ? installations : installations.filter(i => i.companyId === companyId);
            const kpis = [
                { label: 'Clientes Activos', value: clientData.filter(c => c.status === 'Activo').length, icon: 'fa-users', view: 'clients' },
                { label: 'Equipos en Almacén', value: inventoryItems.filter(i => i.status === 'Almacén').length, icon: 'fa-boxes-stacked', view: 'inventory' },
                { label: 'Tareas Pendientes', value: installationData.filter(i => !['Aprobada', 'Finalizada'].includes(i.status)).length, icon: 'fa-tasks', view: 'installations' },
                { label: 'Garantías por Vencer', value: inventoryData.filter(i => calculateWarrantyStatus(i.installationDate, i.warrantyPeriodMonths).class === 'warranty-expiring').length, icon: 'fa-exclamation-triangle', view: 'inventory' }
            ];
            getEl('kpi-cards-container').innerHTML = kpis.map(kpi => `<div class="col-md-6 col-lg-3"><div class="kpi-card" data-view="${kpi.view}"><div class="icon"><i class="fas ${kpi.icon}"></i></div><div class="value">${kpi.value}</div><div class="label">${kpi.label}</div></div></div>`).join('');
        }

        // --- GEOLOCATION MAP ---
        function renderGeolocationMap() {
            let geolocationMap = initMap('geolocation-map', [-34.905, -56.16], 13);
            if (!geolocationMap) return;
            let geolocationMarkerCluster = L.markerClusterGroup();
            clients.forEach(client => {
                if (client.lat && client.lon) {
                    let color = client.status === 'Inactivo' ? 'var(--status-danger)' : 'var(--status-ok)';
                    const icon = L.divIcon({ className: 'custom-div-icon', html: `<div style="background-color: ${color};"><i class="fas fa-user"></i></div>` });
                    geolocationMarkerCluster.addLayer(L.marker([client.lat, client.lon], { icon }).bindPopup(`<b>${client.fullName}</b><br>${client.address}`));
                }
            });
            geolocationMap.addLayer(geolocationMarkerCluster);
        }

        // --- MODAL & FORM INJECTION ---
        function injectModals() {
            getEl('modals-container').innerHTML = `
                <div class="modal fade" id="confirmDeleteModal"><div class="modal-dialog modal-sm modal-dialog-centered"><div class="modal-content"><div class="modal-header"><h5 class="modal-title">Confirmar</h5></div><div class="modal-body">¿Está seguro?</div><div class="modal-footer"><button class="btn btn-secondary" data-bs-dismiss="modal">Cancelar</button><button class="btn btn-danger" id="confirmDeleteBtn">Eliminar</button></div></div></div></div>
                <div class="modal fade" id="detailModal" tabindex="-1"><div class="modal-dialog modal-xl modal-dialog-centered modal-dialog-scrollable"><div class="modal-content"><div class="modal-header"><h5 class="modal-title" id="detailModalTitle"></h5><button class="btn-close btn-close-white" data-bs-dismiss="modal"></button></div><div class="modal-body" id="detailModalBody"></div><div class="modal-footer" id="detailModalFooter"></div></div></div></div>
                <div class="modal fade" id="wizardModal" tabindex="-1"><div class="modal-dialog modal-lg modal-dialog-centered"><div class="modal-content"><div class="modal-header"><h5 class="modal-title" id="wizardModalTitle"></h5><button class="btn-close btn-close-white" data-bs-dismiss="modal"></button></div><div class="modal-body" id="wizardModalBody"></div><div class="modal-footer" id="wizardModalFooter"></div></div></div></div>
                <div class="modal fade" id="installationTaskModal"><div class="modal-dialog modal-xl modal-dialog-centered modal-dialog-scrollable"><div class="modal-content"><div class="modal-header"><h5 class="modal-title" id="installationTaskModalTitle"></h5><button class="btn-close btn-close-white" data-bs-dismiss="modal"></button></div><div class="modal-body" id="installationTaskModalBody"></div><div class="modal-footer" id="installationTaskModalFooter"></div></div></div></div>
                <!-- --- NUEVO --- Modal para solicitud de materiales -->
                <div class="modal fade" id="materialRequestModal" tabindex="-1"><div class="modal-dialog modal-lg modal-dialog-centered"><div class="modal-content"><div class="modal-header"><h5 class="modal-title" id="materialRequestModalTitle">Solicitud de Materiales</h5><button class="btn-close btn-close-white" data-bs-dismiss="modal"></button></div><div class="modal-body" id="materialRequestModalBody"></div><div class="modal-footer"><button class="btn btn-secondary" data-bs-dismiss="modal">Cancelar</button><button class="btn btn-primary" id="sendMaterialRequestBtn">Enviar Solicitud</button></div></div></div></div>
            `;
        }
        
        // --- TABLE RENDERING ---
        let companiesMap = {};
        let clientsMap = {};
        let usersMap = {};
        
        const buildLookups = () => {
            companiesMap = companies.reduce((map, obj) => (map[obj.id] = obj.name, map), {});
            clientsMap = clients.reduce((map, obj) => (map[obj.id] = obj.fullName, map), {});
            usersMap = users.reduce((map, obj) => (map[obj.id] = obj.fullName, map), {});
        };

        const renderTable = (tbodyId, data, rowTemplate) => {
            const tbody = getEl(tbodyId);
            if (tbody) tbody.innerHTML = data.map(rowTemplate).join('');
        }
        const renderCompaniesTable = () => { buildLookups(); renderTable('companiesTableBody', companies, c => `<tr data-id="${c.id}" data-type="company"><td>${c.id}</td><td>${c.name}</td><td>${c.contact}</td><td>${c.email}</td></tr>`); };
        const renderClientsTable = () => { buildLookups(); const data = currentUser.role === 'Superadmin' ? clients : clients.filter(c => c.companyId === currentUser.companyId); renderTable('clientsTableBody', data, cl => `<tr data-id="${cl.id}" data-type="client"><td>${cl.id}</td><td>${cl.fullName}</td><td>${companiesMap[cl.companyId] || 'N/A'}</td><td>${contracts.find(c=>c.clientId === cl.id)?.type || 'Sin Contrato'}</td><td><span class="badge badge-status-${cl.status.toLowerCase()}">${cl.status}</span></td></tr>`); };
        const renderUsersTable = () => { buildLookups(); renderTable('usersTableBody', users, u => `<tr data-id="${u.id}" data-type="user"><td>${u.id}</td><td>${u.username}</td><td>${u.fullName}</td><td>${u.role}</td><td>${companiesMap[u.companyId] || 'N/A'}</td></tr>`); };
        const renderProvidersTable = () => renderTable('providersTableBody', providers, p => `<tr data-id="${p.id}" data-type="provider"><td>${p.id}</td><td>${p.name}</td><td>${p.contact}</td></tr>`);
        const renderServicesTable = () => { buildLookups(); renderTable('servicesTableBody', services, s => `<tr data-id="${s.id}" data-type="service"><td>${s.id}</td><td>${clientsMap[s.clientId]}</td><td>${s.type}</td><td><span class="badge badge-status-${s.status.toLowerCase().replace(' ', '-')}">${s.status}</span></td></tr>`); };
        const renderContractsTable = () => { buildLookups(); renderTable('contractsTableBody', contracts, c => `<tr data-id="${c.id}" data-type="contract"><td>${clientsMap[c.clientId] || 'N/A'}</td><td>${c.type}</td><td>${c.endDate}</td><td><span class="badge badge-status-${c.status.toLowerCase()}">${c.status}</span></td></tr>`); };
        const renderSubcontractsTable = () => renderTable('subcontractsTableBody', subcontracts, sc => {
            const assigned = installations.filter(i => i.subcontractorId === sc.id).length;
            const completed = installations.filter(i => i.subcontractorId === sc.id && i.status === 'Aprobada').length;
            return `<tr data-id="${sc.id}" data-type="subcontract"><td><img src="${sc.contactPhoto}" class="item-thumbnail rounded-circle"></td><td>${sc.contractor}</td><td>${assigned}</td><td>${completed}</td><td><span class="badge badge-status-${sc.status.toLowerCase()}">${sc.status}</span></td></tr>`
        });
        
        // --- INVENTORY VIEW LOGIC ---
        const debounce = (func, delay) => { let timeout; return function(...args) { clearTimeout(timeout); timeout = setTimeout(() => func.apply(this, args), delay); }; };
        
        function renderInventoryView() {
            const data = currentUser.role === 'Superadmin' ? inventoryItems : inventoryItems.filter(i => { const client = clients.find(c => c.id === i.clientId); return (client && client.companyId === currentUser.companyId) || i.status === 'Almacén'; });
            
            const kpis = [ { label: 'Total Artículos', value: data.length, icon: 'fa-boxes-stacked' }, { label: 'En Almacén', value: data.filter(i => i.status === 'Almacén').length, icon: 'fa-warehouse' }, { label: 'Instalados', value: data.filter(i => i.status === 'Instalado').length, icon: 'fa-tools' }, { label: 'En Reparación', value: data.filter(i => i.status === 'Reparación').length, icon: 'fa-screwdriver-wrench' } ];
            getEl('inventory-kpi-cards').innerHTML = kpis.map(kpi => `<div class="col-md-3"><div class="kpi-card p-3"><div class="icon fs-4"><i class="fas ${kpi.icon}"></i></div><div class="value fs-4">${kpi.value}</div><div class="label fs-6">${kpi.label}</div></div></div>`).join('');

            const ctx = getEl('inventoryChart').getContext('2d');
            const statusCounts = data.reduce((acc, item) => { acc[item.status] = (acc[item.status] || 0) + 1; return acc; }, {});
            if (inventoryChartInstance) inventoryChartInstance.destroy();
            inventoryChartInstance = new Chart(ctx, { type: 'doughnut', data: { labels: Object.keys(statusCounts), datasets: [{ data: Object.values(statusCounts), backgroundColor: ['var(--status-warn)', 'var(--status-ok)', 'var(--status-info)', 'var(--status-danger)', 'var(--status-special)'], borderColor: 'var(--mid-bg-color)', borderWidth: 2 }] }, options: { responsive: true, maintainAspectRatio: false, plugins: { legend: { position: 'bottom', labels: { color: 'var(--text-color)' } } } } });

            const searchInput = getEl('inventory-search');
            const applyFilters = () => {
                const searchTerm = searchInput.value.toLowerCase();
                const filteredData = data.filter(item => item.serial.toLowerCase().includes(searchTerm) || item.label.toLowerCase().includes(searchTerm));
                renderInventoryTable(filteredData);
            };
            searchInput.oninput = debounce(applyFilters, 300);
            applyFilters();
        }
        function renderInventoryTable(data = inventoryItems) {
            buildLookups();
            renderTable('inventoryTableBody', data, item => {
                const warranty = calculateWarrantyStatus(item.installationDate, item.warrantyPeriodMonths);
                const locationInfo = item.status === 'Almacén' ? `<strong><i class="fas fa-warehouse me-2"></i>${item.location}</strong>` : `<i class="fas fa-user me-2"></i>${clientsMap[item.clientId] || 'N/A'}`;
                return `<tr data-id="${item.id}" data-type="inventory"><td><img src="${item.thumbnail}" class="item-thumbnail"></td><td><strong>${item.serial}</strong></td><td>${item.label}<br><small class="text-muted">${item.brand} ${item.model}</small></td><td><span class="badge badge-status-${item.status.toLowerCase().replace(/ /g, '-')}">${item.status}</span></td><td>${locationInfo}</td><td class="${warranty.class}"><i class="fas fa-shield-alt me-2"></i>${warranty.text}</td></tr>`;
            });
        }
        const calculateWarrantyStatus = (d, m) => {
            if (!d || !m) return { text: "N/A", class: "" };
            const endDate = new Date(new Date(d).setMonth(new Date(d).getMonth() + m));
            const today = new Date();
            const thirtyDaysFromNow = new Date(new Date().setDate(today.getDate() + 30));
            if (today > endDate) return { text: "Expirada", class: "warranty-expired" };
            if (endDate <= thirtyDaysFromNow) return { text: "Por Vencer", class: "warranty-expiring" };
            return { text: "Activa", class: "warranty-active" };
        };

        // --- INSTALLATIONS VIEW LOGIC ---
        function renderInstallationsView() {
            const data = currentUser.role === 'Superadmin' ? installations : installations.filter(i => i.companyId === currentUser.companyId);
            renderInstallationsTable(data);
        }
        function renderInstallationsTable(data = installations) {
            buildLookups();
            // --- MODIFICADO --- Se añade un icono para solicitudes de material pendientes
            renderTable('installationsTableBody', data, i => `<tr data-id="${i.id}" data-type="installation"><td>${i.id}</td><td>${clientsMap[i.clientId]}</td><td>${i.date}</td><td>${usersMap[i.installerId] || 'Sin Asignar'}</td><td><span class="badge badge-status-${i.status.toLowerCase().replace(/ /g, '-')}">${i.status}</span></td><td>${i.materialRequest && i.materialRequest.status === 'pendiente' ? '<i class="fas fa-wrench text-warning" title="Solicitud de material pendiente"></i>' : ''}</td></tr>`);
        }
        
        const modalConfig = {
            company: { title: "Empresa", data: companies, fields: [ { name: 'name', label: 'Nombre', required: true }, { name: 'nif', label: 'NIF/CIF' }, { name: 'address', label: 'Dirección' }, { name: 'phone', label: 'Teléfono' }, { name: 'email', label: 'Email', type: 'email' }, { name: 'contact', label: 'Contacto' }, { name: 'lat', label: 'Latitud', type: 'number', step: 'any' }, { name: 'lon', label: 'Longitud', type: 'number', step: 'any' } ], onSave: (data, id) => { if (id === 'new') { data.id = generateId(companies); companies.push(data); showToast('Empresa creada.'); } else { const i = companies.findIndex(c => c.id == id); if(i > -1) companies[i] = { ...companies[i], ...data }; showToast('Empresa actualizada.'); } renderCompaniesTable(); }, onDelete: (id) => { companies = companies.filter(c => c.id != id); renderCompaniesTable(); showToast('Empresa eliminada.', 'danger'); } },
            client: { title: "Cliente", data: clients, fields: [ { name: 'fullName', label: 'Nombre Completo', required: true }, { name: 'dni', label: 'DNI/NIF' }, { name: 'companyId', label: 'Empresa', type: 'select', options: () => companies.map(c => ({ value: c.id, text: c.name })) }, { name: 'address', label: 'Dirección' }, { name: 'phone', label: 'Teléfono' }, { name: 'email', label: 'Email', type: 'email' }, { name: 'status', label: 'Estado', type: 'select', options: [{value: 'Activo', text: 'Activo'}, {value: 'Inactivo', text: 'Inactivo'}] }, { name: 'lat', label: 'Latitud', type: 'number', step: 'any' }, { name: 'lon', label: 'Longitud', type: 'number', step: 'any' }, { name: 'notes', label: 'Notas', type: 'textarea' } ], onSave: (data, id) => { data.companyId = parseInt(data.companyId); data.lat = data.lat ? parseFloat(data.lat) : null; data.lon = data.lon ? parseFloat(data.lon) : null; if (id === 'new') { data.id = generateId(clients); clients.push(data); showToast('Cliente creado.'); return data; } else { const i = clients.findIndex(c => c.id == id); if(i > -1) clients[i] = { ...clients[i], ...data }; showToast('Cliente actualizado.'); return clients[i]; } }, onAfterSave: () => { renderClientsTable(); if (currentView === 'geolocation') renderGeolocationMap(); }, onDelete: (id) => { clients = clients.filter(c => c.id != id); renderClientsTable(); if (currentView === 'geolocation') renderGeolocationMap(); showToast('Cliente eliminado.', 'danger'); } },
            inventory: { title: "Artículo de Inventario", data: inventoryItems, fields: [ { name: 'serial', label: 'Serial', required: true }, { name: 'label', label: 'Descripción', required: true }, { name: 'type', label: 'Tipo' }, { name: 'brand', label: 'Marca' }, { name: 'model', label: 'Modelo' }, { name: 'status', label: 'Estado', type: 'select', options: ['Almacén', 'Instalado', 'Arrendado', 'Reparación', 'Vendido', 'Baja'].map(s => ({value: s, text: s}))}, { name: 'location', label: 'Ubicación (si Almacén)'}, { name: 'clientId', label: 'Cliente Asignado', type: 'select', options: () => [{value: '', text: 'Ninguno'}].concat(clients.map(c => ({value: c.id, text: c.fullName})))}, { name: 'warrantyPeriodMonths', label: 'Garantía (meses)', type: 'number' } ], onSave: (data, id) => { data.clientId = data.clientId ? parseInt(data.clientId) : null; data.warrantyPeriodMonths = data.warrantyPeriodMonths ? parseInt(data.warrantyPeriodMonths) : null; if (id === 'new') { data.id = generateId(inventoryItems); data.thumbnail = `https://source.unsplash.com/random/150x150/?${data.type || 'tech'}`; data.history = []; inventoryItems.push(data); addHistoryEvent(data.serial, "Creación", "Artículo registrado."); showToast('Artículo creado.'); } else { const i = inventoryItems.findIndex(item => item.id == id); if(i > -1) { inventoryItems[i] = { ...inventoryItems[i], ...data }; addHistoryEvent(inventoryItems[i].serial, "Actualización", "Datos modificados."); showToast('Artículo actualizado.'); } } renderInventoryView(); }, onDelete: (id) => { const item = inventoryItems.find(i => i.id == id); if(item) addHistoryEvent(item.serial, "Eliminación", "Artículo eliminado."); inventoryItems = inventoryItems.filter(i => i.id != id); renderInventoryView(); showToast('Artículo eliminado.', 'danger'); } },
            installation: { title: "Tarea de Instalación", data: installations, fields: [ { name: 'clientId', label: 'Cliente', type: 'select', options: () => clients.map(c => ({value: c.id, text: c.fullName}))}, { name: 'date', label: 'Fecha', type: 'date' }, { name: 'description', label: 'Descripción', type: 'textarea' }, { name: 'requiredEquipment', label: 'Equipos Requeridos (seriales separados por coma)', type: 'textarea' } ], onSave: (data, id) => { data.clientId = parseInt(data.clientId); data.requiredEquipment = Array.isArray(data.requiredEquipment) ? data.requiredEquipment : data.requiredEquipment.split(',').map(s => s.trim()); if (id === 'new') { data.id = generateId(installations); data.status = 'Pendiente'; data.companyId = clients.find(c => c.id === data.clientId)?.companyId; data.equipmentSerials = []; data.photoUrls = []; data.completionDate = null; data.materialRequest = null; installations.push(data); showToast('Tarea creada.'); } else { const i = installations.findIndex(item => item.id == id); if(i > -1) installations[i] = { ...installations[i], ...data }; showToast('Tarea actualizada.'); } renderInstallationsView(); }, onDelete: (id) => { installations = installations.filter(i => i.id != id); renderInstallationsView(); showToast('Tarea eliminada.', 'danger'); } }
        };

        // --- WIZARD MODAL LOGIC ---
        function showWizardModal(type) { /* ... (Sin cambios, se mantiene igual) ... */ }

        // --- DETAIL MODAL & WORKFLOW ---
        function showDetailModal(type, id) { /* ... (Sin cambios, se mantiene igual) ... */ }
        function showInventoryDetailModal(id) { /* ... (Sin cambios, se mantiene igual) ... */ }

        // --- NUEVO --- Modal para la solicitud de materiales del instalador
        function showMaterialRequestModal(taskId) {
            const task = installations.find(t => t.id == taskId);
            if (!task) return;

            const modal = new bootstrap.Modal(getEl('materialRequestModal'));
            getEl('materialRequestModalTitle').innerText = `Solicitud de Materiales - Tarea #${task.id}`;
            getEl('materialRequestModalBody').innerHTML = `
                <div class="mb-3">
                    <label class="form-label">Materiales Asignados Actualmente</label>
                    <p class="form-control-plaintext ps-2">${task.requiredEquipment.join(', ') || 'Ninguno'}</p>
                </div>
                <div class="mb-3">
                    <label for="materialRequestText" class="form-label">Escriba su solicitud de cambio o adición de materiales</label>
                    <textarea class="form-control" id="materialRequestText" rows="5" placeholder="Ej: Necesito cambiar la cámara CAM-002 por una de visión nocturna. El DVR asignado no es compatible."></textarea>
                </div>
            `;
            
            getEl('sendMaterialRequestBtn').onclick = () => {
                const requestText = getEl('materialRequestText').value;
                if(requestText.trim()) {
                    task.materialRequest = {
                        text: requestText,
                        status: 'pendiente', // 'pendiente', 'aprobada', 'rechazada'
                        date: new Date().toISOString().split('T')[0]
                    };
                    showToast('Solicitud de materiales enviada.');
                    modal.hide();
                    renderInstallerDashboard(currentUser); // Refrescar vista por si hay indicadores
                } else {
                    showToast('Debe escribir una solicitud.', 'danger');
                }
            };

            modal.show();
        }

        // --- MODIFICADO --- Modal de instalación ahora es mucho más complejo y dinámico
        function showInstallationModal(taskId) {
            const task = installations.find(t => t.id == taskId);
            const client = clients.find(c => c.id == task.clientId);
            if (!task || !client) return;
            const modal = new bootstrap.Modal(getEl('installationTaskModal'));
            const isManager = currentUser.role === 'Empresa' || currentUser.role === 'Superadmin';

            const renderContent = () => {
                getEl('installationTaskModalTitle').innerText = `Tarea #${task.id} - ${client.fullName}`;
                let managerControls = '';
                if(isManager) {
                    const installers = users.filter(u => u.role === 'Instalador' && (currentUser.role === 'Superadmin' || u.companyId === currentUser.companyId));
                    managerControls = `
                        <hr><h5 class="mt-4"><i class="fas fa-user-shield me-2"></i>Gestión de Tarea</h5>
                        <div class="row">
                            <div class="col-md-6 mb-3">
                                <label class="form-label">Asignar/Reasignar a</label>
                                <select class="form-select" id="managerInstallerAssignSelect">
                                    <option value="">Sin Asignar</option>
                                    ${installers.map(i => `<option value="${i.id}" ${task.installerId == i.id ? 'selected' : ''}>${i.fullName}</option>`).join('')}
                                </select>
                            </div>
                            <div class="col-md-12 mb-3">
                                <label class="form-label">Modificar Descripción</label>
                                <textarea class="form-control" id="managerDescriptionEdit" rows="3">${task.description}</textarea>
                            </div>
                            <div class="col-md-12 mb-3">
                                <label class="form-label">Modificar Equipos Requeridos (separados por coma)</label>
                                <textarea class="form-control" id="managerRequiredEquipmentEdit" rows="2">${task.requiredEquipment.join(', ')}</textarea>
                            </div>
                        </div>
                    `;
                }

                let materialRequestInfo = '';
                if (task.materialRequest) {
                     materialRequestInfo = `
                        <div class="material-request-box mt-4">
                            <h5><i class="fas fa-wrench me-2"></i>Solicitud de Materiales (${task.materialRequest.status})</h5>
                            <p><strong>Fecha:</strong> ${task.materialRequest.date}</p>
                            <p class="mb-2"><em>"${task.materialRequest.text}"</em></p>
                            ${isManager && task.materialRequest.status === 'pendiente' ? `
                                <button class="btn btn-sm btn-success" id="approveMaterialRequestBtn">Aprobar</button>
                                <button class="btn btn-sm btn-danger" id="rejectMaterialRequestBtn">Rechazar</button>
                            ` : ''}
                        </div>
                     `;
                }

                getEl('installationTaskModalBody').innerHTML = `
                    <div class="row g-4"><div class="col-lg-6">
                        <h5><i class="fas fa-info-circle me-2"></i>Detalles</h5>
                        <p><strong>Dirección:</strong> ${client.address}</p>
                        <p><strong>Descripción:</strong> ${!isManager ? task.description : '<i>Ver sección de gestión</i>'}</p>
                        <h6><strong>Equipos Requeridos:</strong> ${!isManager ? task.requiredEquipment.join(', ') : '<i>Ver sección de gestión</i>'}</h6>
                        ${materialRequestInfo}
                        ${managerControls}
                    </div><div class="col-lg-6">
                        <h5><i class="fas fa-camera me-2"></i>Fotos</h5>
                        <button class="btn btn-outline-primary w-100 mb-3" id="addPhotoBtn"><i class="fas fa-camera-retro me-2"></i>Añadir Foto</button>
                        <input type="file" id="photoInput" accept="image/*" capture="environment" class="d-none">
                        <div id="installation-photos-preview">${task.photoUrls.map((url, i) => `<div class="photo-thumbnail"><img src="${url}"><button class="remove-photo" data-index="${i}">×</button></div>`).join('')}</div><hr>
                        <h5><i class="fas fa-boxes-stacked me-2"></i>Equipos Instalados (Serial)</h5>
                        <div class="input-group mb-3">
                            <input type="text" class="form-control" id="equipmentSerialInput" placeholder="Escanear o escribir serial...">
                            <button class="btn btn-primary" id="addEquipmentBtn"><i class="fas fa-plus"></i></button>
                        </div>
                        <ul class="list-group" id="installedEquipmentList">${task.equipmentSerials.map(s => `<li class="list-group-item d-flex justify-content-between align-items-center bg-transparent">${s} <button class="btn btn-sm btn-outline-danger remove-equipment-btn" data-serial="${s}">×</button></li>`).join('')}</ul>
                        <hr><h5><i class="fas fa-sticky-note me-2"></i>Notas</h5>
                        <textarea class="form-control" id="installationNotes" rows="3">${task.notes}</textarea>
                    </div></div>`;
                
                // Event Listeners for modal content
                getEl('addPhotoBtn').onclick = () => getEl('photoInput').click();
                getEl('photoInput').onchange = e => { if (e.target.files[0]) { const reader = new FileReader(); reader.onload = fr => { task.photoUrls.push(fr.target.result); renderContent(); }; reader.readAsDataURL(e.target.files[0]); } };
                qSA('.remove-photo').forEach(btn => btn.onclick = e => { task.photoUrls.splice(e.target.dataset.index, 1); renderContent(); });
                
                const addEquipment = () => { /* ... (Sin cambios) ... */ };
                getEl('addEquipmentBtn').onclick = addEquipment;
                getEl('equipmentSerialInput').onkeydown = e => { if (e.key === 'Enter') addEquipment(); };
                
                qSA('.remove-equipment-btn').forEach(btn => btn.onclick = e => { /* ... (Sin cambios) ... */ });
                getEl('installationNotes').onchange = e => task.notes = e.target.value;

                if (isManager && task.materialRequest && task.materialRequest.status === 'pendiente') {
                    getEl('approveMaterialRequestBtn').onclick = () => { task.materialRequest.status = 'aprobada'; showToast('Solicitud aprobada.'); renderContent(); renderInstallationsTable(installations); };
                    getEl('rejectMaterialRequestBtn').onclick = () => { task.materialRequest.status = 'rechazada'; showToast('Solicitud rechazada.'); renderContent(); renderInstallationsTable(installations); };
                }
            };
            
            const footer = getEl('installationTaskModalFooter');
            footer.innerHTML = `<button class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>`; // Base button

            if (currentUser.role === 'Instalador' && task.status === 'Asignada') {
                footer.innerHTML += `<button class="btn btn-success ms-auto" id="completeInstallationBtn"><i class="fas fa-check-circle me-2"></i>Marcar como Finalizada</button>`;
                getEl('completeInstallationBtn').onclick = () => { task.status = 'En Revisión'; renderInstallerDashboard(currentUser); modal.hide(); showToast(`Tarea #${task.id} enviada a revisión.`); };
            } else if (isManager) {
                if (task.status === 'En Revisión') {
                    footer.innerHTML += `<button class="btn btn-success ms-auto" id="approveInstallationBtn"><i class="fas fa-thumbs-up me-2"></i>Aprobar Trabajo</button>`;
                    getEl('approveInstallationBtn').onclick = () => { 
                        task.status = 'Aprobada'; 
                        task.completionDate = new Date().toISOString().split('T')[0];
                        renderInstallationsView(); 
                        modal.hide(); 
                        showToast(`Tarea #${task.id} aprobada.`); 
                    };
                }
                footer.innerHTML += `<button class="btn btn-primary ms-2" id="managerSaveChangesBtn">Guardar Cambios</button>`;
                getEl('managerSaveChangesBtn').onclick = () => {
                    task.installerId = parseInt(getEl('managerInstallerAssignSelect').value) || null;
                    task.description = getEl('managerDescriptionEdit').value;
                    task.requiredEquipment = getEl('managerRequiredEquipmentEdit').value.split(',').map(s => s.trim());
                    if(task.installerId && task.status === 'Pendiente') task.status = 'Asignada';
                    showToast('Cambios en la tarea guardados.');
                    renderInstallationsView();
                    modal.hide();
                };
            }

            renderContent();
            modal.show();
        }

        // --- MODIFICADO --- Dashboard de instalador ahora renderiza una tabla
        function renderInstallerDashboard(installer) {
            const tasks = installations.filter(i => i.installerId === installer.id);
            const tableBody = getEl('installer-tasks-table-body');
            
            if (tableBody) {
                tableBody.innerHTML = tasks.length > 0 ? tasks.map(task => {
                    const client = clients.find(c => c.id == task.clientId);
                    return `
                        <tr data-id="${task.id}">
                            <td><span class="badge badge-status-${task.status.toLowerCase().replace(/ /g, '-')}">${task.status}</span></td>
                            <td>${client?.fullName || 'N/A'}</td>
                            <td>${task.date}</td>
                            <td>${task.completionDate || '---'}</td>
                            <td>
                                <button class="btn btn-sm btn-primary view-task-details-btn" title="Ver Detalles"><i class="fas fa-eye"></i></button>
                                ${task.status === 'Asignada' ? `<button class="btn btn-sm btn-warning request-materials-btn" title="Solicitar Materiales"><i class="fas fa-wrench"></i></button>` : ''}
                            </td>
                        </tr>
                    `;
                }).join('') : `<tr><td colspan="5" class="text-center p-4">No tienes tareas asignadas.</td></tr>`;

                // Add event listeners for the new buttons
                qSA('.view-task-details-btn').forEach(btn => btn.onclick = (e) => showInstallationModal(e.target.closest('tr').dataset.id));
                qSA('.request-materials-btn').forEach(btn => btn.onclick = (e) => showMaterialRequestModal(e.target.closest('tr').dataset.id));
            }
        }

        // --- EVENT LISTENERS ---
        function setupEventListeners() {
            getEl('main-app').addEventListener('click', e => {
                const row = e.target.closest('tr[data-id]');
                if (row) {
                    const type = row.dataset.type;
                    const id = row.dataset.id;
                    if (type) {
                        if (type === 'inventory') showInventoryDetailModal(id);
                        else if (type === 'installation') showInstallationModal(id);
                        else if (modalConfig[type]) showDetailModal(type, id);
                    }
                }
            });
            getEl('kpi-cards-container').addEventListener('click', e => { const card = e.target.closest('.kpi-card[data-view]'); if (card) displayView(card.dataset.view); });
            getEl('btn-new-company').addEventListener('click', () => showWizardModal('company'));
            getEl('btn-new-client').addEventListener('click', () => showWizardModal('client'));
            getEl('btn-new-inventory').addEventListener('click', () => showInventoryDetailModal('new'));
            getEl('btn-new-installation').addEventListener('click', () => showDetailModal('installation', 'new'));
        }
        
        // --- INITIALIZATION ---
        document.addEventListener('DOMContentLoaded', () => {
            applyTheme(localStorage.getItem('inventariOS-theme') || 'dark');
            inventoryItems.forEach(item => { if(!item.history || item.history.length === 0) addHistoryEvent(item.serial, "Sistema", "Historial inicializado."); });
            buildLookups();
            injectModals();
            setupEventListeners();
        });
        
    })();
    </script>
</body>
</html>
