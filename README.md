<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>inventariOS v2.21 - WMS Final</title>
    <!-- Dependencias CSS -->
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
    <link href="https://cdn.jsdelivr.net/npm/bootstrap-dark-5/dist/css/bootstrap-dark.min.css" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.5.2/css/all.min.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css">
    <link rel="stylesheet" href="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.css" />
    <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700;800&display=swap" rel="stylesheet">

    <style>
        :root {
            --bg-color: #212529;
            --mid-bg-color: #343a40;
            --light-bg-color: #495057;
            --text-color: #f8f9fa;
            --subtle-text-color: #adb5bd;
            --border-color: rgba(255, 255, 255, 0.1);
            --primary-color: #1abc9c;
            --primary-dark-color: #16a085;
            --glass-bg: rgba(52, 58, 64, 0.8);
            --glass-blur: 18px;
            --input-bg: rgba(0, 0, 0, 0.3);
            --input-border: rgba(255, 255, 255, 0.2);
            --box-shadow-medium: 0 8px 25px rgba(0,0,0,.45);
            --status-ok: #2ecc71;
            --status-warn: #f39c12;
            --status-danger: #e74c3c;
            --status-info: #3498db;
            --status-special: #9b59b6;
        }

        body {
            font-family: 'Poppins', sans-serif;
            background: var(--bg-color);
            color: var(--text-color);
            transition: background-color 0.4s ease, color 0.4s ease;
            overflow: hidden; /* Evitar scroll en el body */
        }

        .background-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            background: linear-gradient(135deg, var(--bg-color) 0%, var(--mid-bg-color) 100%);
            z-index: -1;
        }

        .content-wrapper { 
            position: absolute; 
            top: 0;
            left: 0; 
            width: 100%; 
            height: 100vh; 
            padding-top: 65px; 
            box-sizing: border-box; 
            overflow-y: auto; 
            z-index: 10; 
            scrollbar-width: thin; 
            scrollbar-color: var(--primary-color) var(--mid-bg-color); 
        }
        .content-wrapper.map-active { 
            padding-top: 0; 
            overflow-y: hidden; 
            z-index: 5; 
        }

        .content-wrapper::-webkit-scrollbar {
            width: 8px;
        }

        .content-wrapper::-webkit-scrollbar-track {
            background: var(--mid-bg-color);
        }

        .content-wrapper::-webkit-scrollbar-thumb {
            background-color: var(--primary-color);
            border-radius: 6px;
        }

        .view-section { 
            background-color: var(--glass-bg);
            backdrop-filter: blur(var(--glass-blur));
            -webkit-backdrop-filter: blur(var(--glass-blur));
            padding: 30px;
            border-radius: 15px;
            box-shadow: var(--box-shadow-medium);
            margin: 20px auto;
            width: 95%;
            max-width: 1600px;
            border: 1px solid var(--border-color);
            opacity: 0;
            transform: translateY(20px);
            transition: opacity 0.5s ease-out, transform 0.5s ease-out;
            display: none;
            position: relative; 
            z-index: 10; 
        }

        .view-section.active {
            display: block;
            opacity: 1;
            transform: translateY(0);
        }

        #geolocation-view.view-section, #monitor-view.view-section { 
            padding: 0; 
            margin: 0 auto; 
            border-radius: 0; 
            border: none; 
            box-shadow: none; 
            background-color: transparent; 
            height: calc(100vh - 65px); 
            width: 100%; 
            max-width: 100%; 
            position: relative; 
            z-index: 1; 
        }

        #geolocation-map, #monitor-map { 
            width: 100%;
            height: 100%;
            border-radius: 0;
        }

        .modal-map {
            width: 100%;
            height: 300px;
            border-radius: 8px;
            margin-bottom: 15px;
        }
        #client-map-modal-container {
            height: 100%;
            display: flex;
            flex-direction: column;
        }

        .navbar {
            position: sticky;
            top: 0;
            width: 100%;
            background-color: var(--glass-bg);
            backdrop-filter: blur(var(--glass-blur));
            box-shadow: 0 4px 12px rgba(0, 0, 0, .3);
            z-index: 1000;
            border-bottom: 1px solid var(--border-color);
        }

        .navbar-nav .nav-link,
        .dropdown-item {
            color: var(--text-color) !important; 
        }
        .navbar-nav .nav-link:hover,
        .dropdown-item:hover,
        .navbar-nav .nav-link.active,
        .dropdown-item.active {
            color: var(--primary-color) !important; 
        }


        .section-header {
            display: flex;
            flex-wrap: wrap;
            justify-content: space-between;
            align-items: center;
            gap: 1rem;
            margin-bottom: 2rem;
            border-bottom: 1px solid var(--border-color);
            padding-bottom: 1rem;
        }

        .section-header h2 {
            color: var(--text-color); 
            margin: 0;
            font-weight: 700;
            font-size: 2rem;
        }

        .section-header .search-and-actions {
            display: flex;
            align-items: center;
            gap: 1rem;
        }
        
        .header-actions-group {
            display: flex;
            gap: 0.5rem; 
            flex-wrap: wrap; 
        }
        .header-actions-group .btn {
            white-space: nowrap; 
        }
        
        /* R: Ya no ocultamos los inputs de búsqueda individuales */
        /* .section-header .form-control[placeholder*="Buscar"] {
            display: none;
        } */

        #monitor-controls {
            position: absolute;
            top: 20px;
            right: 20px; 
            left: auto; 
            z-index: 1001;
            display: flex;
            gap: 10px; 
        }

        .table {
            --bs-table-bg: transparent;
            --bs-table-color: var(--text-color);
            --bs-table-border-color: var(--border-color);
            --bs-table-hover-bg: rgba(255, 255, 255, 0.075);
            --bs-table-hover-color: var(--text-color);
        }

        /* R: Estilos para filas de tabla de pagos */
        .table-row-status-pendiente { background-color: rgba(243, 156, 18, 0.15) !important; } /* warn */
        .table-row-status-vencido { background-color: rgba(231, 76, 60, 0.15) !important; } /* danger */
        .table-row-status-pagado { background-color: rgba(46, 204, 113, 0.1) !important; } /* ok */
        /* Para historial de pagos sin estado explícito, un gris claro */
        .table-row-status-registrado { background-color: rgba(149, 165, 166, 0.05) !important; } 

        .table-hover>tbody>tr:hover {
            cursor: pointer;
        }

        .item-thumbnail {
            width: 45px;
            height: 45px;
            object-fit: cover;
            border-radius: 8px;
            margin-right: 15px;
            border: 2px solid var(--border-color);
        }

        .modal.fade .modal-dialog {
            transition: transform .4s ease-out;
            transform: scale(0.9) translateY(-20px);
        }

        .modal.show .modal-dialog {
            transform: scale(1) translateY(0);
        }

        .modal-content {
            border-radius: 15px;
            box-shadow: var(--box-shadow-medium);
            border: none;
            background-color: var(--glass-bg);
            color: var(--text-color);
            backdrop-filter: blur(var(--glass-blur));
        }

        .modal-header {
            color: var(--primary-color);
            border-bottom-color: var(--border-color);
        }

        .modal-body .form-label {
            color: var(--primary-color);
            font-weight: 500;
        }

        .modal-body .form-control,
        .modal-body .form-select {
            background-color: var(--input-bg);
            border: 1px solid var(--input-border);
            color: var(--text-color);
            border-radius: 8px;
        }

        .nav-tabs {
            border-bottom-color: var(--border-color);
        }

        .nav-tabs .nav-link {
            color: var(--subtle-text-color);
            border-color: transparent;
        }

        .nav-tabs .nav-link.active {
            color: var(--primary-color);
            background-color: rgba(0, 0, 0, 0.2);
            border-color: var(--border-color) var(--border-color) transparent;
        }

        .tab-content {
            background-color: rgba(0, 0, 0, 0.1);
            padding: 1.5rem;
            border: 1px solid var(--border-color);
            border-top: none;
            border-radius: 0 0 .5rem .5rem;
        }

        .toast-container {
            z-index: 2050;
        }

        .toast {
            background-color: var(--light-bg-color);
            color: var(--text-color);
            border: 1px solid var(--border-color);
        }

        .badge {
            font-weight: 500;
            font-size: 0.8rem;
            padding: .4em .7em;
        }

        .badge-status-activo,
        .badge-status-finalizada,
        .badge-status-aprobada,
        .badge-status-operativo,
        .badge-status-recibido,
        .badge-status-cerrado,
        .badge-status-pagado, /* R: Nuevo estado pagado */
        .badge-status-abierto {
            background-color: var(--status-ok);
            color: #1a1a1a;
        }

        .badge-status-almacén,
        .badge-status-arrendado,
        .badge-status-venciendo,
        .badge-status-mantenimiento,
        .badge-status-enviado-a-proveedor,
        .badge-status-ordenada,
        .badge-status-pendiente { 
            background-color: var(--status-warn);
            color: #1a1a1a;
        }

        .badge-status-reparación,
        .badge-status-inactivo,
        .badge-status-fuera-de-servicio,
        .badge-status-vencido { /* R: Nuevo estado vencido */
            background-color: var(--status-danger);
            color: var(--text-color);
        }

        .badge-status-baja,
        .badge-status-vendido {
            background-color: #95a5a6;
            color: var(--text-color);
        }

        .badge-status-en-progreso,
        .badge-status-asignada,
        .badge-status-en-revisión,
        .badge-status-iniciado {
            background-color: var(--status-info);
            color: var(--text-color);
        }

        .badge-status-relevamiento,
        .badge-status-planificación {
            background-color: var(--status-special);
            color: var(--text-color);
        }

        .kpi-card {
            background: var(--light-bg-color);
            border-radius: 15px;
            padding: 20px;
            text-align: center;
            transition: all 0.3s ease;
            border: 1px solid var(--border-color);
            cursor: pointer;
        }

        .kpi-card:hover {
            transform: translateY(-10px);
            box-shadow: var(--box-shadow-medium);
            border-color: var(--primary-color);
        }

        .kpi-card .icon {
            font-size: 2.5rem;
            margin-bottom: 10px;
            color: var(--primary-color);
        }

        .kpi-card .value {
            font-size: 2.5rem;
            font-weight: 700;
        }

        .kpi-card .label {
            font-size: 1rem;
            color: var(--subtle-text-color);
        }

        #login-screen {
            display: flex;
            height: 100vh;
            width: 100vw;
            position: fixed;
            top: 0;
            left: 0;
            z-index: 2000;
            background-color: var(--bg-color);
            transition: opacity 0.8s ease-in-out, visibility 0.8s;
            opacity: 1;
            visibility: visible;
        }

        #login-screen.hidden {
            opacity: 0;
            visibility: hidden;
            pointer-events: none;
        }

        .login-image-half {
            flex: 1;
            background-image: linear-gradient(to right, rgba(0, 0, 0, 0.4), var(--bg-color)), url('images/login-image.webp');
            background-size: cover;
            background-position: center;
            animation: fadeInImage 1.2s ease-out;
        }

        @keyframes fadeInImage {
            from {
                opacity: 0;
                transform: scale(1.1);
            }

            to {
                opacity: 1;
                transform: scale(1);
            }
        }

        .login-form-half {
            flex: 1;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 2rem;
            background-color: var(--bg-color);
            animation: slideInForm 1s ease-out forwards;
            opacity: 0;
        }

        @keyframes slideInForm {
            from {
                transform: translateX(50px);
                opacity: 0;
            }

            to {
                transform: translateX(0);
                opacity: 1;
            }
        }

        .login-form-container {
            width: 80%;
            max-width: 450px;
            background: var(--mid-bg-color);
            padding: 3rem;
            border-radius: 12px;
            box-shadow: var(--box-shadow-medium);
            text-align: center;
        }

        .btn .spinner-border {
            width: 1rem;
            height: 1rem;
        }

        .notification-badge {
            position: absolute;
            top: 10px;
            right: 10px;
            font-size: 0.6rem;
        }

        .notification-item {
            font-size: 0.9rem;
            white-space: normal;
        }

        .notification-item small {
            color: var(--subtle-text-color);
        }

        .pagination .page-link {
            background-color: var(--input-bg);
            border-color: var(--input-border);
            color: var(--text-color);
        }

        .pagination .page-item.active .page-link {
            background-color: var(--primary-color);
            border-color: var(--primary-color);
        }

        .table-loading-row td {
            text-align: center;
            padding: 2rem;
            font-style: italic;
            color: var(--subtle-text-color);
        }

        html[data-bs-theme="dark"] .leaflet-control-geocoder {
            border-color: #555;
        }

        .leaflet-control-geocoder,
        .leaflet-control-geocoder-form,
        .leaflet-control-geocoder-icon {
            background-color: var(--input-bg) !important;
            color: var(--text-color) !important;
            border-radius: 8px !important;
        }

        .leaflet-control-geocoder-form input {
            background: transparent !important;
            color: var(--text-color) !important;
        }

        .custom-dot-icon {
            background-color: transparent !important;
            border-radius: 50%;
            text-align: center;
            line-height: 0;
        }

        .custom-dot-icon div {
            width: 16px;
            height: 16px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
            border: 2px solid var(--border-color);
            cursor: grab;
        }

        .ticket-comment {
            border-left: 3px solid var(--primary-color);
        }

        /* R: Buscador de clientes en el mapa - Ajuste de posición */
        #map-client-search-container {
            position: absolute;
            top: 150px; /* Ajustado para moverlo hacia el centro */
            left: 50%;
            transform: translateX(-50%);
            z-index: 1001;
            width: 90%;
            max-width: 500px;
        }

        #map-client-search-input {
            background-color: var(--glass-bg);
            backdrop-filter: blur(var(--glass-blur));
            color: var(--text-color);
            border: 1px solid var(--border-color);
        }

        #map-client-search-results {
            max-height: 300px;
            overflow-y: auto;
            background-color: var(--mid-bg-color);
            border: 1px solid var(--border-color);
            border-top: none;
            border-radius: 0 0 .5rem .5rem;
        }

        #map-client-search-results .list-group-item {
            background-color: transparent;
            color: var(--text-color);
            border-bottom: 1px solid var(--border-color);
            cursor: pointer;
        }

        #map-client-search-results .list-group-item:hover {
            background-color: var(--light-bg-color);
        }

        #map-client-search-results .list-group-item:last-child {
            border-bottom: none;
        }

        .leaflet-popup {
            opacity: 0;
            transform: scale(0.8);
            transition: all 0.2s ease-out;
        }

        .leaflet-popup.leaflet-zoom-animated {
            opacity: 1;
            transform: scale(1);
        }

        .leaflet-popup-content-wrapper,
        .leaflet-popup-tip {
            background: var(--glass-bg);
            backdrop-filter: blur(var(--glass-blur));
            -webkit-backdrop-filter: blur(var(--glass-blur));
            color: var(--text-color);
            border-radius: 12px;
            box-shadow: var(--box-shadow-medium);
            border: 1px solid var(--border-color);
            padding: 15px;
        }

        .leaflet-popup-close-button {
            color: var(--subtle-text-color);
            font-size: 1.5em;
            top: 5px;
            right: 5px;
        }

        .leaflet-popup-close-button:hover {
            color: var(--primary-color);
        }

        .popup-content strong {
            color: var(--primary-color);
        }

        .popup-content small {
            color: var(--subtle-text-color);
        }

        .popup-content p {
            margin-bottom: 0.5rem;
        }

        .comment-entry {
            border-left: 3px solid var(--primary-color);
            background-color: var(--input-bg);
            border: 1px solid var(--input-border);
            border-radius: 8px;
            padding: 10px 15px;
            margin-bottom: 10px;
        }

        .comment-entry strong {
            color: var(--primary-color);
        }

        .comment-entry small {
            color: var(--subtle-text-color);
            font-size: 0.85em;
        }

        .comment-entry p {
            margin-bottom: 0;
            margin-top: 5px;
        }
        
        #clientDetailModal .tab-content, #detailModal .tab-content {
            background-color: transparent; 
            border: none;
            padding: 0; 
        }
        #clientDetailModal .nav-tabs .nav-link.active, #detailModal .nav-tabs .nav-link.active {
            background-color: var(--light-bg-color); 
        }

        .platform-icon {
            font-size: 1.5rem;
            margin-right: 8px;
        }

        .stock-low { color: var(--status-warn) !important; font-weight: bold; }
        .stock-critical { color: var(--status-danger) !important; font-weight: bold; }

    </style>
</head>
<body data-bs-theme="dark">

    <div class="background-overlay"></div>
    <div class="toast-container position-fixed top-0 end-0 p-3"></div>
    
    <div id="login-screen">
        <div class="login-image-half d-none d-md-block"></div>
        <div class="login-form-half">
            <div class="login-form-container">
                <h2>Bienvenido a inventariOS v2.21</h2>
                <form id="loginForm">
                    <div class="input-group mb-3">
                        <label class="input-group-text" for="loginRole"><i class="fas fa-user-shield"></i></label>
                        <select class="form-select form-select-lg" id="loginRole">
                            <option value="Admin">Admin</option>
                            <option value="Empresa">Empresa</option>
                        </select>
                    </div>
                    <div class="mb-3"><input type="text" class="form-control form-control-lg" id="loginUsername" placeholder="Usuario" required></div>
                    <div class="mb-4"><input type="password" class="form-control form-control-lg" id="loginPassword" placeholder="Contraseña" required></div>
                    <button type="submit" class="btn btn-primary btn-lg w-100 mb-3" id="loginButton">
                        <span class="spinner-border spinner-border-sm d-none" role="status" aria-hidden="true"></span>
                        Ingresar
                    </button>
                </form>
                <button id="resetDataBtn" class="btn btn-sm btn-outline-danger mt-3">Resetear Datos de Ejemplo</button>
            </div>
        </div>
    </div>

    <div id="main-app" style="display: none;">
        <nav class="navbar navbar-expand-lg">
            <div class="container-fluid">
                <a class="navbar-brand" href="#" data-view="dashboard"><i class="fas fa-cubes-alt me-2"></i>inventariOS</a>
                <button class="navbar-toggler" type="button" data-bs-toggle="collapse" data-bs-target="#navbarNav"><span class="navbar-toggler-icon"></span></button>
                <div class="collapse navbar-collapse" id="navbarNav">
                    <ul class="navbar-nav me-auto" id="main-nav-links"></ul>
                    <ul class="navbar-nav ms-auto d-flex align-items-center">
                        <li class="nav-item dropdown">
                            <a class="nav-link" href="#" id="notificationBell" role="button" data-bs-toggle="dropdown" aria-expanded="false"><i class="fas fa-bell"></i><span class="badge rounded-pill bg-danger notification-badge d-none" id="notificationCount"></span></a>
                            <ul class="dropdown-menu dropdown-menu-end" aria-labelledby="notificationBell" id="notificationList"></ul>
                        </li>
                        <li class="nav-item dropdown">
                            <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-bs-toggle="dropdown"><i class="fas fa-user-circle me-1"></i><span id="navbarUsernameDisplay">Cargando...</span></a>
                            <ul class="dropdown-menu dropdown-menu-end">
                                <li><a class="dropdown-item" href="#" id="logoutBtnNav"><i class="fas fa-sign-out-alt me-2"></i>Cerrar Sesión</a></li>
                            </ul>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>

        <!-- R: El buscador de mapa se mantiene independiente del content-wrapper -->
        <div id="map-client-search-container" class="d-none">
            <input type="text" id="map-client-search-input" class="form-control form-control-lg mb-0" placeholder="Buscar cliente en mapa (Esc para cerrar)...">
            <div id="map-client-search-results" class="list-group">
                <!-- Los resultados se llenarán con JS -->
            </div>
        </div>
        <!-- Fin de buscador de mapa -->

        <div class="content-wrapper" id="main-content-wrapper">
            <div id="dashboard-view" class="view-section">
                <div class="section-header"><h2><i class="fas fa-tachometer-alt me-2"></i>Dashboard</h2></div>
                <div class="row g-4">
                    <div class="col-lg-12">
                        <div id="kpi-cards-container" class="row g-4"></div>
                    </div>
                </div>
            </div>
            
            <div id="geolocation-view" class="view-section">
                <div id="geolocation-map"></div>
            </div> 
            
            <div id="monitor-view" class="view-section">
                <div id="monitor-map"></div>
                <!-- R: Controles del monitor de pagos -->
                <div id="monitor-controls">
                    <!-- <button class="btn btn-primary me-2" id="btn-add-monitor-device"><i class="fas fa-plus me-2"></i>Agregar Dispositivo</button> -->
                    <select id="monitor-filter-status" class="form-select d-inline-block" style="width: auto;">
                        <option value="">Todos los Estados</option>
                        <option value="Pendiente">Pendiente</option>
                        <option value="Pagado">Pagado</option>
                        <option value="Vencido">Vencido</option>
                    </select>
                </div>
            </div>

            <!-- R: companies-view eliminada según el nuevo modelo de datos -->

            <div id="clients-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-users me-2"></i>Clientes</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-client"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-clients"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-clients" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-clients"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover align-middle" id="clientsTable"><thead><tr><th></th><th>ID</th><th>Nombre</th><th>Contrato</th><th>Monto Mensual</th><th>Estado</th></tr></thead><tbody id="clientsTableBody"></tbody></table>
                </div>
            </div>
            <div id="users-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-user-shield me-2"></i>Usuarios</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-user"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-users"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-users" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-users"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="usersTable"><thead><tr><th>ID</th><th>Usuario</th><th>Nombre</th><th>Rol</th><th>Empresa/Cliente Asignado</th></tr></thead><tbody id="usersTableBody"></tbody></table>
                </div>
            </div>
            <div id="providers-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-truck me-2"></i>Proveedores</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-provider"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-providers"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-providers" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-providers"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="providersTable"><thead><tr><th>ID</th><th>Nombre</th><th>Contacto</th><th>Teléfono</th><th>Email</th></tr></thead><tbody id="providersTableBody"></tbody></table>
                </div>
            </div>
            <div id="supports-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-headset me-2"></i>Soportes</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-support"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-supports"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-supports" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-supports"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="supportsTable"><thead><tr><th>ID</th><th>Título</th><th>Asociado a</th><th>Responsable</th><th>Estado</th><th>Comentarios</th><th>Mapa</th></tr></thead><tbody id="supportsTableBody"></tbody></table>
                </div>
            </div>
            <div id="fleet-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-car me-2"></i>Flota Vehicular</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-fleet"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-fleet"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-fleet" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-fleet"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover align-middle" id="fleetTable"><thead><tr><th></th><th>Matrícula</th><th>Marca</th><th>Modelo</th><th>Responsable</th><th>Estado</th></tr></thead><tbody id="fleetTableBody"></tbody></table>
                </div>
            </div>
            <div id="warehouses-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-warehouse me-2"></i>Almacenes</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-warehouse"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-warehouses"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-warehouses" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-warehouses"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="warehousesTable"><thead><tr><th>ID</th><th>Nombre</th><th>Ubicación</th><th>Responsable</th></tr></thead><tbody id="warehousesTableBody"></tbody></table>
                </div>
            </div>

            <div id="products-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-tag me-2"></i>Catálogo de Productos</h2>
                    <div class="search-and-actions">
                        <input type="text" id="products-search" class="form-control me-2" placeholder="Buscar producto...">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-product"><i class="fas fa-plus me-2"></i>Nuevo</button>
                            <button class="btn btn-secondary" id="btn-export-products"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-products" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-products"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="productsTable"><thead><tr><th>ID</th><th>Descripción</th><th>Tipo</th><th>Marca</th><th>Modelo</th><th>Stock Total</th><th>Stock Mín.</th></tr></thead><tbody id="productsTableBody"></tbody></table>
                </div>
            </div>

            <div id="inventory-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-boxes-stacked me-2"></i>Inventario de Ítems</h2>
                    <div class="search-and-actions">
                        <input type="text" id="inventory-search" class="form-control me-2" placeholder="Buscar por Serial...">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-inventory-item"><i class="fas fa-plus me-2"></i>Nuevo Ítem</button>
                            <button class="btn btn-secondary" id="btn-export-inventoryItems"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-inventoryItems" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-inventoryItems"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover align-middle" id="inventoryItemsTable"><thead><tr><th>ID</th><th>Serial</th><th>Producto</th><th>Estado</th><th>Ubicación</th></tr></thead><tbody id="inventoryItemsTableBody"></tbody></table>
                </div>
                <nav id="inventory-pagination" class="mt-4 d-flex justify-content-center"></nav>
            </div>

            <div id="inventory-movements-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-people-carry me-2"></i>Movimientos de Inventario</h2>
                    <div class="search-and-actions">
                        <input type="text" id="movements-search" class="form-control me-2" placeholder="Buscar movimiento...">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-movement"><i class="fas fa-plus me-2"></i>Nuevo Movimiento</button>
                            <button class="btn btn-secondary" id="btn-export-inventoryMovements"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-inventoryMovements" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-inventoryMovements"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="inventoryMovementsTable"><thead><tr><th>ID</th><th>Fecha</th><th>Tipo</th><th>Artículo Serial</th><th>De</th><th>A</th><th>Usuario</th></tr></thead><tbody id="inventoryMovementsTableBody"></tbody></table>
                </div>
            </div>

            <div id="contracts-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-file-contract me-2"></i>Contratos</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-primary" id="btn-new-contract"><i class="fas fa-plus me-2"></i>Nuevo Contrato</button>
                            <button class="btn btn-secondary" id="btn-export-contracts"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                            <input type="file" id="file-input-contracts" class="d-none" accept=".csv, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel">
                            <button class="btn btn-outline-secondary" id="btn-import-contracts"><i class="fas fa-file-import me-2"></i>Importar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="contractsTable"><thead><tr><th>ID</th><th>Tipo</th><th>Cliente</th><th>Fecha Inicio</th><th>Fecha Fin</th><th>Estado</th><th>Costo Mensual</th></tr></thead><tbody id="contractsTableBody"></tbody></table>
                </div>
            </div>

            <div id="payments-view" class="view-section">
                <div class="section-header">
                    <h2><i class="fas fa-money-bill-alt me-2"></i>Pagos</h2>
                    <div class="search-and-actions">
                        <div class="header-actions-group">
                            <button class="btn btn-secondary" id="btn-export-payments"><i class="fas fa-file-excel me-2"></i>Exportar</button>
                        </div>
                    </div>
                </div>
                <div class="table-responsive">
                    <table class="table table-dark table-hover" id="paymentsTable"><thead><tr><th>Fecha</th><th>Tipo</th><th>Entidad</th><th>Monto</th><th>Descripción</th><th>Estado</th><th>Acción</th></tr></thead><tbody id="paymentsTableBody"></tbody></table>
                </div>
            </div>

        </div>
    </div>
    
    <div id="modals-container"></div>

    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"></script>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
    <script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>
    <script src="https://unpkg.com/leaflet-control-geocoder/dist/Control.Geocoder.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/qrcode-generator/qrcode.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf-autotable/3.5.23/jspdf.plugin.autotable.min.js"></script>

    <script>
    (function() {
        'use strict';
        try { 

            // --- DATA MODEL (DB SIMULATION) ---
            const AppDB_template = {
                clients: [ 
                    { id: 101, fullName: "Edificio Central Park", address: "Av. Brasil 2525, Montevideo", phone: "099111222", email: "admin@centralpark.uy", notes: "Cliente residencial premium.", lat: -34.9113, lon: -56.1535, status: "Activo", contractType: "Mantenimiento", monthlyAmount: 1500, monthlyCurrency: "UYU", photoUrl: "https://source.unsplash.com/random/150x150/?apartment,building",
                        paymentHistory: [{date: "2024-06-01", amount: 1500, description: "Pago mantenimiento junio"}, {date: "2024-05-01", amount: 1500, description: "Pago mantenimiento mayo"}],
                        facadePhotoUrl: "https://source.unsplash.com/random/400x300/?building,apartment"
                    }, 
                    { id: 102, fullName: "Oficinas World Trade Center", address: "Av. Dr. Luis Alberto de Herrera 1248, Montevideo", phone: "26261234", email: "seguridad@wtc.com.uy", notes: "Requiere equipos de alta gama.", lat: -34.8958, lon: -56.1428, status: "Activo", contractType: "Arriendo", monthlyAmount: 2000, monthlyCurrency: "USD", photoUrl: "https://source.unsplash.com/random/150x150/?skyscraper,office",
                        paymentHistory: [{date: "2024-06-05", amount: 2000, description: "Pago arriendo junio"}],
                        facadePhotoUrl: "https://source.unsplash.com/random/400x300/?office,skyscraper"
                    }, 
                    { id: 103, fullName: "Farmacia Pocitos", address: "Benito Blanco 980, Montevideo", phone: "098333444", email: "farmacia.pocitos@gmail.com", notes: "", lat: -34.9183, lon: -56.1501, status: "Inactivo", contractType: "Sin Contrato", monthlyAmount: 0, monthlyCurrency: "UYU", photoUrl: "https://source.unsplash.com/random/150x150/?pharmacy,store",
                        paymentHistory: [],
                        facadePhotoUrl: "https://source.unsplash.com/random/400x300/?pharmacy"
                    } 
                ],
                users: [ 
                    { id: 1, username: "admin", password: "admin", role: "Admin", fullName: "Admin General", notifications: [] }, 
                    { id: 2, username: "empresa", password: "empresa", role: "Empresa", fullName: "Gerente SecureTech", notifications: [] }, 
                    { id: 3, username: "fgonzalez", password: "fgonzalez", role: "Empresa", fullName: "Fernando Gonzalez", notifications: [] } 
                ],
                providers: [{ id: 1, name: "DistriEquipos S.A.", contact: "Gerardo López", phone: "24041122", email: "ventas@distriequipos.com.uy", paymentType: "Transferencia", hasCredit: true, bankInfo: "BROU C/A 123-456789", purchaseHistory: [{date: "2024-05-10", item: "Lote Cámaras", amount: 2500}] }],
                supports: [{id: 1, title: "Fallo en cámara de acceso", subject: "Fallo de cámara", details: "El cliente reporta que la cámara de acceso dejó de funcionar.", associatedToType: "Cliente", associatedToId: 102, managerId: 2, status: "Abierto", comments: [{user: "Admin General", date: "2024-07-01T10:00:00", text: "Ticket abierto. Se asigna a Pedro."}, {user: "Pedro", date: "2024-07-01T11:00:00", text: "Revisando conexión en sitio."}], attachedFiles: ["doc1.pdf", "img1.jpg"]},
                           {id: 2, title: "Problema de red en oficina", subject: "Sin conectividad en la oficina", details: "El cliente reporta que no hay internet en su oficina principal.", associatedToType: "Cliente", associatedToId: 101, managerId: 1, status: "En Progreso", comments: [{user: "Admin General", date: "2024-07-02T09:00:00", text: "Problema reportado."}], attachedFiles: []}],
                fleet: [{id: "SBC1234", brand: "Fiat", model: "Fiorino", responsibleId: 2, status: "Operativo", photoUrl: "https://source.unsplash.com/random/150x150/?van,white", 
                    fines: [{date: "2024-06-20", reason: "Exceso de velocidad", amount: 1500}],
                    repairs: [{date: "2024-05-15", description: "Cambio de aceite", cost: 800, doneBy: "Taller A"}]
                }],
                warehouses: [{id: 1, name: "Almacén Central", location: "Calle Falsa 123", responsibleId: 1}, {id: 2, name: "Depósito Sucursal Este", location: "Av. Italia 5555", responsibleId: 2}],
                products: [
                    {id: 1, label: "Cámara Domo 4MP", type: "Cámara IP", brand: "Hikvision", model: "DS-2CD2143G0-I", minStock: 5},
                    {id: 2, label: "DVR 16 Canales", type: "DVR", brand: "Hikvision", model: "iDS-7216HQHI-M2/S", minStock: 2},
                    {id: 3, label: "Panel de Alarma AX Pro", type: "Alarma", brand: "Hikvision", model: "DS-PWA96-M-WE", minStock: 10}
                ],
                inventoryItems: [
                    { id: 1001, productId: 1, serial: "CAM-001", status: "Instalado", locationType: "Client", locationId: 101, installationDate: "2023-01-15", notes: "Instalado en entrada principal."},
                    { id: 1002, productId: 2, serial: "DVR-001", status: "Instalado", locationType: "Client", locationId: 101, installationDate: "2023-01-15", notes: "Grabador en sala de seguridad."},
                    { id: 1003, productId: 1, serial: "CAM-002", status: "Almacén", locationType: "Warehouse", locationId: 1, installationDate: null, notes: "En estante 3A."},
                    { id: 1004, productId: 3, serial: "ALM-001", status: "Almacén", locationType: "Warehouse", locationId: 1, installationDate: null, notes: "Para próxima instalación."},
                    { id: 1005, productId: 1, serial: "CAM-003", status: "En Reparación", locationType: "Provider", locationId: 1, installationDate: "2024-03-01", notes: "Enviado a DistriEquipos por falla de lente."}
                ],
                inventoryMovements: [
                    { id: 1, date: "2023-01-10", type: "Entrada", inventoryItemId: 1001, fromLocationType: "Provider", fromLocationId: 1, toLocationType: "Warehouse", toLocationId: 1, userId: 1 },
                    { id: 2, date: "2023-01-15", type: "Salida", inventoryItemId: 1001, fromLocationType: "Warehouse", fromLocationId: 1, toLocationType: "Client", toLocationId: 101, userId: 2 },
                    { id: 3, date: "2024-03-01", type: "Transferencia", inventoryItemId: 1005, fromLocationType: "Warehouse", fromLocationId: 1, toLocationType: "Provider", toLocationId: 1, userId: 1 }
                ],
                // R: Nuevo modelo para Recordatorios de Pagos (antes monitorDevices)
                paymentReminders: [ 
                    { id: 1, clientId: 101, dueDate: "2024-07-28", amount: 1500, currency: "UYU", status: "Pendiente", description: "Factura mantenimiento Julio" },
                    { id: 2, clientId: 102, dueDate: "2024-07-20", amount: 2000, currency: "USD", status: "Vencido", description: "Factura arriendo Julio" },
                    { id: 3, clientId: 103, dueDate: "2024-07-05", amount: 0, currency: "UYU", status: "Pagado", description: "Factura mantenimiento Junio" }
                ],
                contracts: [
                    { id: 1, clientId: 101, type: "Mantenimiento", startDate: "2023-01-01", endDate: "2025-01-01", status: "Activo", monthlyFee: 1500, currency: "UYU", terms: "Contrato de mantenimiento preventivo y correctivo." },
                    { id: 2, clientId: 102, type: "Arriendo", startDate: "2023-03-01", endDate: "2024-09-30", status: "Activo", monthlyFee: 2000, currency: "USD", terms: "Contrato de arrendamiento de equipos de seguridad." }
                ],
                auditLog: []
            };
            
            const DB_KEY = 'inventariOS_DB';
            // R: Se fuerza la carga de la plantilla si la DB no existe o si quieres reiniciar para ver los cambios
            let AppDB_Live = JSON.parse(localStorage.getItem(DB_KEY)) || JSON.parse(JSON.stringify(AppDB_template));
            
            const saveDB = () => {
                localStorage.setItem(DB_KEY, JSON.stringify(AppDB_Live));
            };

            const API = { 
                _call(fn) { 
                    const DELAY = Math.random() * 20 + 5; 
                    return new Promise(resolve => { setTimeout(() => resolve(fn()), DELAY); }); 
                }, 
                login(username, password, role) { 
                    return this._call(() => { 
                        const user = AppDB_Live.users.find(u => u.username.toLowerCase() === username.toLowerCase() && u.role.toLowerCase() === role.toLowerCase()); 
                        if (!user || user.password !== password) {
                            throw new Error("Credenciales incorrectas o rol no válido."); 
                        }
                        return { ...user }; 
                    }); 
                }, 
                _createApiFor(entityName) { 
                    const db = AppDB_Live[entityName]; 
                    if (!db || !Array.isArray(db)) { 
                        console.warn(`API: La entidad '${entityName}' no existe o no es un array en AppDB_Live.`);
                        return { findAll: () => Promise.resolve([]), findById: () => Promise.resolve(null), create: () => Promise.reject(new Error("Entidad no existe")), update: () => Promise.reject(new Error("Entidad no existe")), delete: () => Promise.reject(new Error("Entidad no existe")) };
                    }
                    const entitySingular = entityName.endsWith('ies') ? entityName.slice(0, -3) + 'y' : entityName.slice(0, -1); 
                    return { 
                        findAll: (filterFn = () => true) => this._call(() => db.filter(filterFn).map(item => ({...item}))), 
                        findById: (id) => this._call(() => { const item = db.find(item => item.id == id); return item ? {...item} : null; }), 
                        create: (data) => this._call(async () => { 
                            const newId = db.length > 0 ? Math.max(0, ...db.map(i => i.id || 0)) + 1 : 1; 
                            const newItem = { ...data, id: newId }; 
                            db.push(newItem); 
                            saveDB(); 
                            if (currentUser) AppDB_Live.auditLog.unshift({ date: new Date().toISOString(), user: currentUser.username, action: `CREATED ${entitySingular} #${newId}` }); 
                            if (entityName === 'supports' && newItem.managerId) {
                                await addNotification(newItem.managerId, `Se te ha asignado un nuevo soporte: ${newItem.subject} (#${newId})`);
                            }
                            return { ...newItem }; 
                        }), 
                        update: (id, data) => this._call(async () => { 
                            const index = db.findIndex(item => item.id == id); 
                            if (index === -1) return null; 
                            const oldItem = { ...db[index] }; 
                            db[index] = { ...db[index], ...data }; 
                            saveDB(); 
                            if (currentUser) AppDB_Live.auditLog.unshift({ date: new Date().toISOString(), user: currentUser.username, action: `UPDATED ${entitySingular} #${id}` }); 
                            if (entityName === 'supports' && oldItem.managerId != db[index].managerId && db[index].managerId) {
                                await addNotification(db[index].managerId, `Se te ha reasignado el soporte: ${db[index].subject} (#${id}). Nuevo estado: ${db[index].status}`);
                            }
                            if (entityName === 'supports' && oldItem.status !== db[index].status) {
                                let messageForAssociated = '';
                                if (db[index].associatedToType === 'Cliente' && db[index].associatedToId) {
                                    const client = allClientsData.find(c => c.id == db[index].associatedToId);
                                    if (client) {
                                        messageForAssociated = `El soporte "${db[index].subject}" (#${id}) para ${client.fullName} ha cambiado de estado a: ${db[index].status}.`;
                                    }
                                } 
                            }
                            return { ...db[index] }; 
                        }), 
                        delete: (id) => this._call(() => { 
                            const index = db.findIndex(item => item.id == id); 
                            if (index === -1) return false; 
                            if (currentUser) AppDB_Live.auditLog.unshift({ date: new Date().toISOString(), user: currentUser.username, action: `DELETED ${entitySingular} #${id}` }); 
                            db.splice(index, 1); 
                            saveDB(); 
                            return true; 
                        }) 
                    }; 
                }, 
                init() { 
                    Object.keys(AppDB_Live).forEach(key => { 
                        if(Array.isArray(AppDB_Live[key])) { 
                            this[key] = this._createApiFor(key); 
                        } 
                    }); 
                } 
            };
            API.init();
            
            let currentUser = null; 
            let currentView = 'dashboard'; 
            let lookups = {};
            let geolocationMapInstance = null; 
            // R: Renombrado monitorMapInstance a paymentMonitorMapInstance
            let paymentMonitorMapInstance = null; 
            let mapClientSearchTimeout = null; 
            let allClientsData = []; 
            // R: allCompaniesData eliminado
            let clientMapInstance = null; 
            // R: monitorDevicesData renombrado a paymentRemindersData
            let paymentRemindersData = []; 

            const getEl = (id) => document.getElementById(id); 
            const qS = (selector) => document.querySelector(selector); 
            const qSA = (selector) => document.querySelectorAll(selector);
            
            const showToast = (message, type = 'success') => { 
                const toastId = `toast-${Date.now()}`; 
                const toastHeaderClass = type === 'success' ? 'text-success' : type === 'danger' ? 'text-danger' : 'text-warning'; 
                const toastIcon = type === 'success' ? 'fa-check-circle' : type === 'danger' ? 'fa-times-circle' : 'fa-exclamation-triangle'; 
                const toastHTML = `<div id="${toastId}" class="toast" role="alert" aria-live="assertive" aria-atomic="true"><div class="toast-header"><i class="fas ${toastIcon} ${toastHeaderClass} me-2"></i><strong class="me-auto">Notificación</strong><button type="button" class="btn-close btn-close-white" data-bs-dismiss="toast"></button></div><div class="toast-body">${message}</div></div>`; 
                getEl('toast-container')?.insertAdjacentHTML('beforeend', toastHTML); 
                const toastElement = getEl(toastId); 
                if(toastElement) { 
                    const bsToast = new bootstrap.Toast(toastElement); 
                    bsToast.show(); 
                    toastElement.addEventListener('hidden.bs.toast', () => toastElement.remove()); 
                }
            };
            const debounce = (func, delay) => { let timeout; return function(...args) { clearTimeout(timeout); timeout = setTimeout(() => func.apply(this, args), delay); }; };
            const toggleLoading = (element, isLoading) => { if (!element) return; const spinner = element.querySelector('.spinner-border'); if (isLoading) { element.disabled = true; if (spinner) spinner.classList.remove('d-none'); } else { element.disabled = false; if (spinner) spinner.classList.add('d-none'); } };
            
            // R: getDotColor ahora incluye estados de pago
            const getDotColor = (itemStatus, itemContractType = null) => { 
                switch (itemStatus) {
                    case 'Activo': case 'Online': case 'Pagado': return 'var(--status-ok)';
                    case 'Inactivo': case 'Offline': case 'Vencido': case 'De Baja': return 'var(--status-danger)';
                    case 'Advertencia': case 'Pendiente': return 'var(--status-warn)';
                    default:
                        // Lógica para tipos de contrato si aún se usan para otros propósitos en el mapa
                        switch (itemContractType) { 
                            case 'Mantenimiento': return 'var(--status-info)';
                            case 'Arriendo': return 'var(--status-special)';
                            case 'Sin Contrato': return 'var(--subtle-text-color)';
                            default: return '#cccccc'; 
                        }
                }
            };

            const navConfig = { 
                Admin: [ 
                    { view: 'dashboard', icon: 'fa-tachometer-alt', label: 'Dashboard' }, 
                    { view: 'geolocation', icon: 'fa-map-marked-alt', label: 'Mapa' }, 
                    { view: 'monitor', icon: 'fa-globe', label: 'Monitor Pagos' }, // R: Nombre actualizado
                    { view: 'supports', icon: 'fa-headset', label: 'Soportes' }, 
                    { label: 'Gestión', icon: 'fa-sitemap', submenu: [ 
                        { view: 'clients', icon: 'fa-users', label: 'Clientes' }, 
                        { view: 'users', icon: 'fa-user-shield', label: 'Usuarios' }, 
                        { view: 'providers', icon: 'fa-truck', label: 'Proveedores' }, 
                        { view: 'contracts', icon: 'fa-file-contract', label: 'Contratos' } 
                    ] }, 
                    { label: 'Inventario', icon: 'fa-boxes-stacked', submenu: [ 
                        { view: 'products', icon: 'fa-tag', label: 'Catálogo Productos' },
                        { view: 'inventory', icon: 'fa-boxes', label: 'Ítems de Inventario' },
                        { view: 'inventory-movements', icon: 'fa-people-carry', label: 'Movimientos' },
                        { view: 'warehouses', icon: 'fa-warehouse', label: 'Almacenes' } 
                    ]},
                    { label: 'Operaciones', icon: 'fa-cogs', submenu: [ 
                        { view: 'fleet', icon: 'fa-car', label: 'Flota' },
                        { view: 'payments', icon: 'fa-money-bill-alt', label: 'Pagos' } 
                    ] } 
                ], 
                Empresa: [ 
                    { view: 'dashboard', icon: 'fa-tachometer-alt', label: 'Dashboard' }, 
                    { view: 'geolocation', icon: 'fa-map-marked-alt', label: 'Mapa' }, 
                    { view: 'clients', icon: 'fa-users', label: 'Clientes' }, 
                    { view: 'providers', icon: 'fa-truck', label: 'Proveedores' },
                    { label: 'Inventario', icon: 'fa-boxes-stacked', submenu: [ 
                        { view: 'products', icon: 'fa-tag', label: 'Catálogo Productos' },
                        { view: 'inventory', icon: 'fa-boxes', label: 'Ítems de Inventario' },
                        { view: 'inventory-movements', icon: 'fa-people-carry', label: 'Movimientos' },
                        { view: 'warehouses', icon: 'fa-warehouse', label: 'Almacenes' } 
                    ]} 
                ] 
            };
            const renderFunctions = { 
                dashboard: renderDashboard, 
                geolocation: renderGeolocationMap, 
                monitor: renderPaymentMonitorView, // R: Cambiado a renderPaymentMonitorView
                clients: renderClientsTable, 
                users: renderUsersTable, 
                providers: renderProvidersTable, 
                supports: renderSupportsTable, 
                fleet: renderFleetTable, 
                warehouses: renderWarehousesTable, 
                products: renderProductsTable,
                inventory: renderInventoryView,
                'inventory-movements': renderInventoryMovements,
                contracts: renderContractsTable, 
                payments: renderPaymentsTable, 
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
            }
            function displayView(viewId) { 
                if (!currentUser) return; 
                const mainContentWrapper = getEl('main-content-wrapper');
                
                qSA('.view-section').forEach(s => s.classList.remove('active')); 

                if (currentView === viewId) { 
                    if (viewId === 'geolocation' && geolocationMapInstance) {
                        geolocationMapInstance.invalidateSize();
                    } else if (viewId === 'monitor' && paymentMonitorMapInstance) { // R: Cambio de monitorMapInstance
                        paymentMonitorMapInstance.invalidateSize();
                    }
                    return; 
                }
                
                currentView = viewId; 
                qSA('#main-nav-links .nav-link, #main-nav-links .dropdown-item').forEach(l => l.classList.remove('active')); 
                
                const targetView = getEl(`${viewId}-view`); 
                if (targetView) { 
                    targetView.classList.add('active'); 

                    if (viewId === 'geolocation' && geolocationMapInstance) {
                        geolocationMapInstance.invalidateSize();
                    } else if (viewId === 'monitor' && paymentMonitorMapInstance) { // R: Cambio de monitorMapInstance
                        paymentMonitorMapInstance.invalidateSize();
                    }
                }

                const activeLink = qS(`#main-nav-links a[data-view="${viewId}"], .navbar-brand[data-view="${viewId}"]`); 
                if (activeLink) { 
                    activeLink.classList.add('active'); 
                    const dropdown = activeLink.closest('.dropdown'); 
                    if(dropdown) dropdown.querySelector('.dropdown-toggle').classList.add('active'); 
                } 
                if (renderFunctions[viewId]) { 
                    renderFunctions[viewId](); 
                } else { 
                    console.warn(`No render function for view: ${viewId}`); 
                } 
            }
            const buildLookups = async () => { 
                const [clients, users, warehouses, providers, products, inventoryItems, paymentReminders, contracts] = await Promise.all([ // R: Cambiado monitorDevices por paymentReminders
                    API.clients.findAll(), API.users.findAll(), API.warehouses.findAll(), API.providers.findAll(), API.products.findAll(), API.inventoryItems.findAll(), API.paymentReminders.findAll(), API.contracts.findAll()
                ]); 
                lookups.clients = clients.reduce((map, obj) => (map[obj.id] = obj.fullName, map), {}); 
                lookups.users = users.reduce((map, obj) => (map[obj.id] = obj.fullName, map), {}); 
                lookups.warehouses = warehouses.reduce((map, obj) => (map[obj.id] = obj.name, map), {}); 
                lookups.providers = providers.reduce((map, obj) => (map[obj.id] = obj.name, map), {}); 
                lookups.products = products.reduce((map, obj) => (map[obj.id] = obj.label, map), {}); 
                lookups.inventoryItems = inventoryItems.reduce((map, obj) => (map[obj.id] = obj.serial, map), {}); 
                lookups.paymentReminders = paymentReminders.reduce((map, obj) => (map[obj.id] = obj.description, map), {}); // R: Añadido paymentReminders lookup
                lookups.contracts = contracts.reduce((map, obj) => (map[obj.id] = `${obj.type} (#${obj.id})`, map), {}); 

                allClientsData = clients; 
                paymentRemindersData = paymentReminders; // R: Cambiado monitorDevicesData a paymentRemindersData
            };

            const renderNotifications = async () => { 
                const user = await API.users.findById(currentUser.id); 
                const countEl = getEl('notificationCount'); 
                const listEl = getEl('notificationList'); 
                const notifications = (user.notifications || []).filter(n => !n.read); 
                if (notifications.length > 0) { 
                    countEl.textContent = notifications.length; 
                    countEl.classList.remove('d-none'); 
                    listEl.innerHTML = notifications.map(n => `<li><a class="dropdown-item notification-item" href="#"><div>${n.message}</div><small>${new Date(n.date).toLocaleString()}</small></a></li>`).join('') + '<li><hr class="dropdown-divider"></li><li><a class="dropdown-item text-center" href="#" id="mark-all-read">Marcar todas como leídas</a></li>'; 
                    getEl('mark-all-read').onclick = async () => { 
                        const allNotifications = (await API.users.findById(currentUser.id)).notifications; 
                        const updatedNotifications = allNotifications.map(n => ({...n, read: true})); 
                        await API.users.update(currentUser.id, {notifications: updatedNotifications}); 
                        renderNotifications(); 
                    }; 
                } else { 
                    countEl.classList.add('d-none'); 
                    listEl.innerHTML = '<li><p class="text-center text-muted mb-0 p-2">No hay notificaciones nuevas</p></li>'; 
                } 
            };
            const addNotification = async (userId, message) => { 
                const user = await API.users.findById(userId); 
                if (!user) return; 
                const newNotification = { date: new Date().toISOString(), message, read: false }; 
                const currentNotifications = user.notifications || []; 
                await API.users.update(userId, { notifications: [newNotification, ...currentNotifications] }); 
                if (currentUser && currentUser.id === userId) renderNotifications(); 
            };

            async function renderDashboard() {
                const isSuperAdmin = currentUser.role === 'Admin'; 
                const [allClients, allSupports, allFleet, allProducts, allPaymentReminders, allContracts] = await Promise.all([ // R: Cambiado allMonitorDevices a allPaymentReminders
                    API.clients.findAll(), API.supports.findAll(), API.fleet.findAll(), API.products.findAll(), API.paymentReminders.findAll(), API.contracts.findAll() 
                ]);
                const clientData = isSuperAdmin ? allClients : allClients; 
                
                getEl('dashboard-view').innerHTML = `
                    <div class="section-header"><h2><i class="fas fa-tachometer-alt me-2"></i>Dashboard</h2></div>
                    <div class="row g-4">
                        <div class="col-12 col-md-8">
                            <div class="row g-4" id="kpi-cards-container"></div>
                        </div>
                        <div class="col-12 col-md-4">
                            <div class="card h-100 bg-dark text-white border-secondary">
                                <div class="card-header text-primary"><h4>Actividad Reciente</h4></div>
                                <div class="card-body" id="activity-feed" style="max-height: 400px; overflow-y: auto;">
                                    <p class="text-muted text-center">Cargando actividad...</p>
                                </div>
                            </div>
                        </div>
                    </div>
                    <div class="row g-4 mt-4">
                        <div class="col-12">
                             <div class="card bg-dark text-white border-secondary">
                                <div class="card-header text-primary"><h4>Resumen de Inventario</h4></div>
                                <div class="card-body">
                                    <canvas id="inventoryStatusChart"></canvas>
                                </div>
                            </div>
                        </div>
                    </div>
                `;

                // Renderizar KPIs
                const kpis = [ 
                    { label: 'Clientes Activos', value: clientData.filter(c => c.status === 'Activo').length, icon: 'fa-users', view: 'clients' }, 
                    { label: 'Tickets en Progreso', value: allSupports.filter(s => ['En Progreso', 'Abierto'].includes(s.status)).length, icon: 'fa-headset', view: 'supports' },
                    { label: 'Vehículos Operativos', value: allFleet.filter(v => v.status === 'Operativo').length, icon: 'fa-car', view: 'fleet' },
                    { label: 'Contratos Activos', value: allContracts.filter(c => c.status === 'Activo').length, icon: 'fa-file-contract', view: 'contracts' }, 
                    { label: 'Productos Registrados', value: allProducts.length, icon: 'fa-tag', view: 'products' }, 
                    { label: 'Items en Almacén', value: (await API.inventoryItems.findAll(i => i.locationType === 'Warehouse')).length, icon: 'fa-warehouse', view: 'inventory' },
                    { label: 'Pagos Pendientes', value: allPaymentReminders.filter(d => d.status === 'Pendiente' || d.status === 'Vencido').length, icon: 'fa-money-check-alt', view: 'payments' } // R: KPI actualizado
                ];
                getEl('kpi-cards-container').innerHTML = kpis.map(kpi => `<div class="col-md-6 col-lg-4"><div class="kpi-card" data-view="${kpi.view}"><div class="icon"><i class="fas ${kpi.icon}"></i></div><div class="value">${kpi.value}</div><div class="label">${kpi.label}</div></div></div>`).join('');
                
                const inventoryItems = await API.inventoryItems.findAll();
                const products = await API.products.findAll();
                const productStockData = products.map(p => ({
                    label: p.label,
                    stock: inventoryItems.filter(item => item.productId === p.id && item.status !== 'De Baja').length,
                    minStock: p.minStock
                }));

                const ctx = getEl('inventoryStatusChart').getContext('2d');
                const labels = productStockData.map(p => p.label);
                const dataValues = productStockData.map(p => p.stock);
                const backgroundColors = productStockData.map(p => p.stock < p.minStock ? 'var(--status-danger)' : (p.stock < p.minStock * 1.5 ? 'var(--status-warn)' : 'var(--status-ok)'));

                if (window.inventoryStatusChartInstance) window.inventoryStatusChartInstance.destroy();
                window.inventoryStatusChartInstance = new Chart(ctx, {
                    type: 'bar',
                    data: {
                        labels: labels,
                        datasets: [{
                            label: 'Stock Actual',
                            data: dataValues,
                            backgroundColor: backgroundColors,
                        }]
                    },
                    options: {
                        responsive: true,
                        maintainAspectRatio: false,
                        plugins: {
                            legend: { display: false },
                            title: { display: true, text: 'Stock de Productos', color: 'var(--text-color)' }
                        },
                        scales: {
                            y: { beginAtZero: true, ticks: { color: 'var(--subtle-text-color)' }, grid: { color: 'rgba(255,255,255,0.1)' } },
                            x: { ticks: { color: 'var(--subtle-text-color)' }, grid: { display: false } }
                        }
                    }
                });

                const activityFeedEl = getEl('activity-feed');
                if (activityFeedEl) {
                    const auditLog = AppDB_Live.auditLog.slice(0, 10); 
                    activityFeedEl.innerHTML = auditLog.length > 0 ? auditLog.map(log => `
                        <div class="mb-2 border-bottom pb-2">
                            <small class="text-muted">${new Date(log.date).toLocaleString()}</small><br>
                            ${log.user}: <strong>${log.action}</strong>
                        </div>
                    `).join('') : '<p class="text-muted text-center">No hay actividad reciente.</p>';
                }
            }
            
            async function renderGeolocationMap() {
                let mapContainer = getEl('geolocation-map'); 
                if (geolocationMapInstance) { 
                    geolocationMapInstance.remove(); 
                    geolocationMapInstance = null; 
                }

                geolocationMapInstance = L.map('geolocation-map').setView([-34.905, -56.16], 13); 
                const tileUrl = 'https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png'; 
                L.tileLayer(tileUrl, { attribution: '© OpenStreetMap & Stadia Maps', maxZoom: 19 }).addTo(geolocationMapInstance); 

                L.Control.geocoder({ defaultMarkGeocode: false, placeholder: 'Buscar dirección...' }).on('markgeocode', e => geolocationMapInstance.setView(e.geocode.center, 16)).addTo(geolocationMapInstance); 
                
                let markerCluster = L.markerClusterGroup();
                const clients = allClientsData; 

                clients.forEach(client => {
                    if (client.lat && client.lon) {
                        const color = getDotColor(client.status, client.contractType);
                        const icon = L.divIcon({ 
                            className: 'custom-dot-icon', 
                            html: `<div style="background-color: ${color};"></div>`, 
                            iconSize: [24, 24] 
                        });
                        const marker = L.marker([client.lat, client.lon], { icon, draggable: true }).bindPopup(`
                            <div class="popup-content">
                                <p><strong>${client.fullName}</strong></p>
                                <p><small>${client.address}</small></p>
                                <p><small>Estado: ${client.status}</small></p>
                                <p><small>Contrato: ${client.contractType}</small></p>
                                <button class="btn btn-sm btn-primary mt-2" data-client-id="${client.id}" id="viewClientDetailBtn">Ver Detalles</button>
                            </div>
                        `);
                        marker.on('dragend', async (event) => {
                            const newCoords = event.target.getLatLng();
                            await API.clients.update(client.id, { lat: newCoords.lat, lon: newCoords.lng });
                            showToast(`Ubicación de ${client.fullName} actualizada.`);
                            renderGeolocationMap(); 
                        });
                        markerCluster.addLayer(marker);
                        marker.on('popupopen', (e) => {
                            const btn = e.popup.getElement().querySelector('#viewClientDetailBtn'); 
                            if (btn) {
                                btn.onclick = () => {
                                    showClientDetailModal(client.id); 
                                    e.popup.remove(); 
                                };
                            }
                        });
                    }
                });
                geolocationMapInstance.addLayer(markerCluster); 
                geolocationMapInstance.on('click', (e) => { 
                    const searchContainer = getEl('map-client-search-container');
                    if (!searchContainer.classList.contains('d-none')) return; 
                    showToast('Abriendo formulario para nuevo cliente en esta ubicación...', 'info'); 
                    showCrudModal('client', 'new', { lat: e.latlng.lat, lon: e.latlng.lng }); 
                }); 
            }

            // R: Nueva función para el Monitoreo de Pagos
            async function renderPaymentMonitorView() {
                let mapContainer = getEl('monitor-map');
                if (paymentMonitorMapInstance) { // R: Cambio de monitorMapInstance
                    paymentMonitorMapInstance.remove(); // R: Cambio de monitorMapInstance
                    paymentMonitorMapInstance = null; // R: Cambio de monitorMapInstance
                }

                paymentMonitorMapInstance = L.map('monitor-map').setView([-34.905, -56.16], 13); // R: Cambio de monitorMapInstance
                const tileUrl = 'https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png';
                L.tileLayer(tileUrl, { attribution: '© OpenStreetMap & Stadia Maps', maxZoom: 19 }).addTo(paymentMonitorMapInstance); // R: Cambio de monitorMapInstance

                let paymentMarkerCluster = L.markerClusterGroup();
                // R: Obtenemos los recordatorios de pagos y los clientes para sus ubicaciones
                const paymentReminders = await API.paymentReminders.findAll();
                const clients = await API.clients.findAll();

                const selectedStatus = getEl('monitor-filter-status').value;
                const filteredReminders = selectedStatus ? paymentReminders.filter(p => p.status === selectedStatus) : paymentReminders;

                filteredReminders.forEach(reminder => {
                    const client = clients.find(c => c.id === reminder.clientId);
                    if (client && client.lat && client.lon) {
                        const color = getDotColor(reminder.status); 
                        const icon = L.divIcon({ 
                            className: 'custom-dot-icon', 
                            html: `<div style="background-color: ${color};"></div>`, 
                            iconSize: [24, 24] 
                        });
                        let popupContent = `
                            <div class="popup-content">
                                <p><strong>${reminder.description}</strong></p>
                                <p><small>Cliente: ${client.fullName}</small></p>
                                <p><small>Monto: $${reminder.amount} ${reminder.currency}</small></p>
                                <p><small>Vence: ${reminder.dueDate}</small></p>
                                <p><small>Estado: ${reminder.status}</small></p>
                                <button class="btn btn-sm btn-primary mt-2" data-payment-reminder-id="${reminder.id}" id="viewPaymentReminderDetailBtn">Ver Detalles</button>
                            </div>
                        `;
                        const marker = L.marker([client.lat, client.lon], { icon }).bindPopup(popupContent);
                        
                        paymentMarkerCluster.addLayer(marker);

                        marker.on('popupopen', (e) => {
                            const btn = e.popup.getElement().querySelector('#viewPaymentReminderDetailBtn');
                            if (btn) {
                                btn.onclick = () => {
                                    showCrudModal('paymentReminder', reminder.id); 
                                    e.popup.remove();
                                };
                            }
                        });
                    }
                });
                paymentMonitorMapInstance.addLayer(paymentMarkerCluster); // R: Cambio de monitorMapInstance

                // R: El botón de "Agregar Dispositivo" fue eliminado del HTML para esta vista.
                // El filtro ya está asociado con su evento en setupEventListeners.
                const monitorFilterStatus = getEl('monitor-filter-status');
                if (monitorFilterStatus) {
                    monitorFilterStatus.onchange = renderPaymentMonitorView; 
                }

                paymentMonitorMapInstance.invalidateSize(); // R: Cambio de monitorMapInstance
            }

            const renderTable = (tbodyId, data, rowTemplate) => { const tbody = getEl(tbodyId); if (tbody) tbody.innerHTML = data.map(rowTemplate).join('') || `<tr><td colspan="100%" class="text-center p-4">No se encontraron datos.</td></tr>`; };
            
            async function renderClientsTable() { 
                const data = await API.clients.findAll(); 
                renderTable('clientsTableBody', data, cl => `<tr data-id="${cl.id}" data-type="client"><td><img src="${cl.photoUrl}" class="item-thumbnail"></td><td>${cl.id}</td><td>${cl.fullName}</td><td>${cl.contractType || 'N/A'}</td><td>$${cl.monthlyAmount || 0} ${cl.monthlyCurrency || ''}</td><td><span class="badge badge-status-${cl.status.toLowerCase()}">${cl.status}</span></td></tr>`); 
            }
            async function renderUsersTable() { 
                const data = await API.users.findAll(); 
                renderTable('usersTableBody', data, u => `<tr data-id="${u.id}" data-type="user"><td>${u.id}</td><td>${u.username}</td><td>${u.fullName}</td><td>${u.role}</td><td>N/A</td></tr>`); 
            }
            async function renderProvidersTable() { const data = await API.providers.findAll(); renderTable('providersTableBody', data, p => `<tr data-id="${p.id}" data-type="provider"><td>${p.id}</td><td>${p.name}</td><td>${p.contact}</td><td>${p.phone}</td><td>${p.email}</td></tr>`); }
            async function renderSupportsTable() { 
                const filterFn = currentUser.role === 'Admin' ? () => true : (s) => s.managerId == currentUser.id;
                const data = await API.supports.findAll(filterFn); 
                renderTable('supportsTableBody', data, i => `<tr data-id="${i.id}" data-type="support">
                    <td>${i.id}</td>
                    <td>${i.subject}</td> 
                    <td>${lookups.clients[i.associatedToId] || 'N/A'}</td> 
                    <td>${lookups.users[i.managerId] || 'N/A'}</td>
                    <td><span class="badge badge-status-${i.status.toLowerCase().replace(/ /g, '-')}">${i.status}</span></td>
                    <td>${i.comments ? i.comments.length : 0}</td>
                    <td><button class="btn btn-sm btn-outline-info view-on-map-btn" data-support-id="${i.id}"><i class="fas fa-map-marker-alt"></i></button></td> 
                </tr>`); 
                qSA('#supportsTableBody .view-on-map-btn').forEach(btn => {
                    btn.onclick = async (e) => {
                        e.stopPropagation(); 
                        const supportId = e.currentTarget.dataset.supportId;
                        const support = await API.supports.findById(supportId);
                        if (support && support.associatedToType === 'Cliente' && support.associatedToId) {
                            const client = await API.clients.findById(support.associatedToId);
                            if (client && client.lat && client.lon) {
                                displayView('geolocation'); 
                                setTimeout(() => {
                                    if (geolocationMapInstance) {
                                        geolocationMapInstance.flyTo([client.lat, client.lon], 17); 
                                        geolocationMapInstance.eachLayer(layer => {
                                            if (layer instanceof L.Marker && layer.getLatLng().lat === client.lat && layer.getLatLng().lng === client.lon) {
                                                layer.openPopup();
                                            }
                                        });
                                    }
                                }, 500); 
                            } else {
                                showToast('Cliente asociado no tiene ubicación o no encontrado.', 'warning');
                            }
                        } else {
                            showToast('Este soporte no está asociado a un cliente con ubicación.', 'info');
                        }
                    };
                });
            }
            async function renderFleetTable() { const data = await API.fleet.findAll(); renderTable('fleetTableBody', data, v => `<tr data-id="${v.id}" data-type="fleet"><td><img src="${v.photoUrl}" class="item-thumbnail"></td><td>${v.id}</td><td>${v.brand}</td><td>${v.model}</td><td>${lookups.users[v.responsibleId] || 'Sin asignar'}</td><td><span class="badge badge-status-${v.status.toLowerCase().replace(/ /g, '-')}">${v.status}</span></td></tr>`); }
            async function renderWarehousesTable() { const data = await API.warehouses.findAll(); renderTable('warehousesTableBody', data, w => `<tr data-id="${w.id}" data-type="warehouse"><td>${w.id}</td><td>${w.name}</td><td>${w.location}</td><td>${lookups.users[w.responsibleId] || 'N/A'}</td></tr>`); }
            
            async function renderProductsTable() { 
                const data = await API.products.findAll(); 
                renderTable('productsTableBody', data, p => {
                    const totalStock = AppDB_Live.inventoryItems.filter(item => item.productId === p.id && item.status !== 'De Baja').length;
                    const stockClass = totalStock < p.minStock ? 'stock-critical' : (totalStock < p.minStock * 1.5 ? 'stock-low' : '');
                    return `<tr data-id="${p.id}" data-type="product">
                                <td>${p.id}</td>
                                <td>${p.label}</td>
                                <td>${p.type}</td>
                                <td>${p.brand}</td>
                                <td>${p.model}</td>
                                <td class="${stockClass}">${totalStock}</td>
                                <td>${p.minStock}</td>
                            </tr>`;
                }); 
            }

            let inventoryCurrentPage = 1; 
            const ITEMS_PER_PAGE = 10;
            async function renderInventoryView() { 
                applyInventoryFilters(); 
            }
            async function applyInventoryFilters() {
                const tbody = getEl('inventoryItemsTableBody'); 
                if (tbody) tbody.innerHTML = `<tr><td colspan="5" class="table-loading-row"><div class="spinner-border text-primary" role="status"></div></td></tr>`;
                
                const searchTerm = getEl('inventory-search').value.toLowerCase();
                const allItems = await API.inventoryItems.findAll();
                
                const filteredData = allItems.filter(item => {
                    const productLabel = lookups.products[item.productId] || '';
                    return (item.serial && item.serial.toLowerCase().includes(searchTerm)) ||
                           (productLabel.toLowerCase().includes(searchTerm));
                });
                
                renderPaginatedTable('inventoryItemsTableBody', 'inventory-pagination', filteredData, inventoryCurrentPage, ITEMS_PER_PAGE, renderInventoryItemRow, (newPage) => { 
                    inventoryCurrentPage = newPage; applyInventoryFilters(); 
                });
            }
            const renderInventoryItemRow = (item) => { 
                const product = lookups.products[item.productId] || 'N/A';
                let locationInfo = 'N/A';
                if (item.locationType === 'Warehouse' && item.locationId) {
                    locationInfo = lookups.warehouses[item.locationId] || 'Almacén Desconocido';
                } else if (item.locationType === 'Client' && item.locationId) {
                    locationInfo = lookups.clients[item.locationId] || 'Cliente Desconocido';
                } else if (item.locationType === 'Provider' && item.locationId) {
                    locationInfo = lookups.providers[item.locationId] || 'Proveedor Desconocido';
                }
                return `<tr data-id="${item.id}" data-type="inventoryItem">
                            <td>${item.id}</td>
                            <td>${item.serial}</td>
                            <td>${product}</td>
                            <td><span class="badge badge-status-${item.status.toLowerCase().replace(/ /g, '-')}">${item.status}</span></td>
                            <td>${locationInfo}</td>
                        </tr>`; 
            };

            async function renderInventoryMovements() {
                const data = await API.inventoryMovements.findAll();
                renderTable('inventoryMovementsTableBody', data, m => {
                    const itemSerial = lookups.inventoryItems[m.inventoryItemId] || 'N/A';
                    let fromLocation = 'N/A';
                    if (m.fromLocationType === 'Warehouse') fromLocation = lookups.warehouses[m.fromLocationId] || 'Almacén Desconocido';
                    else if (m.fromLocationType === 'Client') fromLocation = lookups.clients[m.fromLocationId] || 'Cliente Desconocido';
                    else if (m.fromLocationType === 'Provider') fromLocation = lookups.providers[m.fromLocationId] || 'Proveedor Desconocido';

                    let toLocation = 'N/A';
                    if (m.toLocationType === 'Warehouse') toLocation = lookups.warehouses[m.toLocationId] || 'Almacén Desconocido';
                    else if (m.toLocationType === 'Client') toLocation = lookups.clients[m.toLocationId] || 'Cliente Desconocido';
                    else if (m.toLocationType === 'Provider') toLocation = lookups.providers[m.toLocationId] || 'Proveedor Desconocido';

                    return `<tr data-id="${m.id}" data-type="inventoryMovement">
                                <td>${m.id}</td>
                                <td>${new Date(m.date).toLocaleDateString()}</td>
                                <td>${m.type}</td>
                                <td>${itemSerial}</td>
                                <td>${fromLocation}</td>
                                <td>${toLocation}</td>
                                <td>${lookups.users[m.userId] || 'N/A'}</td>
                            </tr>`;
                });
            }

            async function renderContractsTable() {
                const data = await API.contracts.findAll(c => currentUser.role === 'Admin' || (c.clientId && allClientsData.find(cl => cl.id === c.clientId)) ); 
                renderTable('contractsTableBody', data, c => `<tr data-id="${c.id}" data-type="contract">
                    <td>${c.id}</td>
                    <td>${c.type}</td>
                    <td>${lookups.clients[c.clientId] || 'N/A'}</td> 
                    <td>${c.startDate}</td>
                    <td>${c.endDate}</td>
                    <td><span class="badge badge-status-${c.status.toLowerCase()}">${c.status}</span></td>
                    <td>$${c.monthlyFee || 0} ${c.currency || ''}</td>
                </tr>`);
            }

            // R: Pagos (vista global combinada de historial y recordatorios)
            async function renderPaymentsTable() {
                let allPaymentsAndReminders = [];

                const clients = await API.clients.findAll();
                clients.forEach(client => {
                    (client.paymentHistory || []).forEach(payment => {
                        allPaymentsAndReminders.push({
                            id: `history-${client.id}-${payment.date}-${payment.amount}`, 
                            type: 'Pago Registrado',
                            entityName: client.fullName,
                            entityId: client.id,
                            date: payment.date,
                            amount: payment.amount,
                            currency: client.monthlyCurrency || 'N/A', 
                            description: payment.description,
                            status: 'Pagado', // Historial de pagos siempre es 'Pagado'
                            isReminder: false // Marcador para diferenciar en la tabla
                        });
                    });
                });

                const paymentReminders = await API.paymentReminders.findAll();
                paymentReminders.forEach(reminder => {
                    const client = clients.find(c => c.id === reminder.clientId);
                    allPaymentsAndReminders.push({
                        id: `reminder-${reminder.id}`, 
                        type: 'Recordatorio',
                        entityName: client ? client.fullName : 'Cliente Desconocido',
                        entityId: reminder.clientId,
                        date: reminder.dueDate, // Usar dueDate para el orden y visualización
                        amount: reminder.amount,
                        currency: reminder.currency,
                        description: reminder.description,
                        status: reminder.status,
                        isReminder: true,
                        reminderId: reminder.id // Guardar ID del recordatorio original
                    });
                });

                allPaymentsAndReminders.sort((a, b) => new Date(b.date).getTime() - new Date(a.date).getTime());

                renderTable('paymentsTableBody', allPaymentsAndReminders, p => {
                    const statusClass = p.status ? `table-row-status-${p.status.toLowerCase().replace(/ /g, '-')}` : 'table-row-status-registrado';
                    const badgeHtml = p.status ? `<span class="badge badge-status-${p.status.toLowerCase().replace(/ /g, '-')}">${p.status}</span>` : '';
                    
                    let actionButton = '';
                    if (p.isReminder && (p.status === 'Pendiente' || p.status === 'Vencido')) {
                        actionButton = `<button class="btn btn-sm btn-success register-payment-btn" data-reminder-id="${p.reminderId}" data-client-id="${p.entityId}" data-amount="${p.amount}" data-currency="${p.currency}" data-description="${p.description}">Registrar Pago</button>`;
                    }

                    return `<tr data-id="${p.id}" data-type="payment" class="${statusClass}">
                        <td>${new Date(p.date).toLocaleDateString()}</td>
                        <td>${p.type}</td>
                        <td>${p.entityName} (ID: ${p.entityId})</td>
                        <td>$${p.amount} ${p.currency}</td>
                        <td>${p.description}</td>
                        <td>${badgeHtml}</td>
                        <td>${actionButton}</td>
                    </tr>`;
                });

                // R: Adjuntar event listeners para los botones "Registrar Pago"
                qSA('#paymentsTableBody .register-payment-btn').forEach(button => {
                    button.onclick = async (e) => {
                        const reminderId = parseInt(e.currentTarget.dataset.reminderId, 10);
                        const clientId = parseInt(e.currentTarget.dataset.clientId, 10);
                        const amount = parseFloat(e.currentTarget.dataset.amount);
                        const currency = e.currentTarget.dataset.currency;
                        const description = e.currentTarget.dataset.description;

                        if (confirm(`¿Confirma que desea registrar el pago de $${amount} ${currency} para ${lookups.clients[clientId]}?`)) {
                            // 1. Actualizar el estado del recordatorio a 'Pagado'
                            await API.paymentReminders.update(reminderId, { status: 'Pagado' });
                            
                            // 2. Añadir la entrada al historial de pagos del cliente
                            const client = await API.clients.findById(clientId);
                            if (client) {
                                const newPaymentEntry = {
                                    date: new Date().toISOString().split('T')[0],
                                    amount: amount,
                                    description: `Pago de ${description} (registrado desde recordatorio)`
                                };
                                const updatedPaymentHistory = [...(client.paymentHistory || []), newPaymentEntry];
                                await API.clients.update(clientId, { paymentHistory: updatedPaymentHistory });
                                showToast('Pago registrado y recordatorio actualizado.', 'success');
                            } else {
                                showToast('Error: Cliente no encontrado para registrar historial de pago.', 'danger');
                            }
                            await buildLookups(); // Reconstruir lookups para actualizar datos
                            renderPaymentsTable(); // Refrescar la tabla de pagos
                            renderPaymentMonitorView(); // Refrescar el mapa de monitoreo de pagos
                        }
                    };
                });
            }


            function renderPaginatedTable(tbodyId, navId, data, currentPage, perPage, rowTemplate, onPageClick) {
                const tbody = getEl(tbodyId); const nav = getEl(navId); const totalPages = Math.ceil(data.length / perPage); const start = (currentPage - 1) * perPage; const end = start + perPage; const paginatedData = data.slice(start, end);
                tbody.innerHTML = paginatedData.map(rowTemplate).join('') || `<tr><td colspan="100%" class="text-center p-4">No se encontraron resultados.</td></tr>`;
                let paginationHTML = '';
                if (totalPages > 1) { paginationHTML = `<ul class="pagination">`; paginationHTML += `<li class="page-item ${currentPage === 1 ? 'disabled' : ''}"><a class="page-link" href="#" data-page="${currentPage - 1}">Anterior</a></li>`; for (let i = 1; i <= totalPages; i++) { paginationHTML += `<li class="page-item ${i === currentPage ? 'active' : ''}"><a class="page-link" href="#" data-page="${i}">${i}</a></li>`; } paginationHTML += `<li class="page-item ${currentPage === totalPages ? 'disabled' : ''}"><a class="page-link" href="#" data-page="${currentPage + 1}">Siguiente</a></li>`; paginationHTML += `</ul>`; }
                nav.innerHTML = paginationHTML;
                qSA(`#${navId} a.page-link`).forEach(link => { link.addEventListener('click', (e) => { e.preventDefault(); if (!link.parentElement.classList.contains('disabled')) { onPageClick(parseInt(e.target.dataset.page)); } }); });
            }

            const modalConfig = {
                client: { 
                    title: "Cliente", 
                    fields: [ 
                        { name: 'fullName', label: 'Nombre Completo', required: true }, 
                        { name: 'contractType', label: 'Tipo de Contrato', type: 'select', options: ['Sin Contrato', 'Mantenimiento', 'Arriendo'] }, 
                        { name: 'address', label: 'Dirección' }, 
                        { name: 'phone', label: 'Teléfono', type: 'tel' }, 
                        { name: 'email', label: 'Email', type: 'email' }, 
                        { name: 'status', label: 'Estado', type: 'select', options: ['Activo', 'Inactivo'] }, 
                        { name: 'monthlyAmount', label: 'Monto Mensual', type: 'number', step: '0.01' }, 
                        { name: 'photoUrl', label: 'URL de Foto' }, 
                        { name: 'lat', label: 'Latitud', type: 'number', step: 'any' }, 
                        { name: 'lon', label: 'Longitud', type: 'number', step: 'any' }, 
                        { name: 'notes', label: 'Notas', type: 'textarea' } 
                    ], 
                    api: API.clients, 
                    onFinish: async () => { await buildLookups(); await renderClientsTable(); if(currentView === 'geolocation') await renderGeolocationMap(); } 
                },
                user: { title: "Usuario", fields: [ { name: 'fullName', label: 'Nombre Completo', required: true }, { name: 'username', label: 'Nombre de Usuario', required: true }, { name: 'password', label: 'Contraseña', type: 'password', isNewOnly: true }, { name: 'role', label: 'Rol', type: 'select', options: ['Admin', 'Empresa'] }, 
                    ], api: API.users, onFinish: renderUsersTable },
                provider: { title: "Proveedor", fields: [ { name: 'name', label: 'Nombre', required: true }, { name: 'contact', label: 'Contacto' }, { name: 'phone', label: 'Teléfono', type: 'tel' }, { name: 'email', type: 'email' }, { name: 'paymentType', label: 'Tipo de Pago', type: 'select', options: ['Efectivo', 'Transferencia', 'Cheque', 'Crédito'] }, { name: 'hasCredit', label: 'Tiene Crédito', type: 'select', options: [{value: true, text: 'Sí'}, {value: false, text: 'No'}] }, { name: 'bankInfo', label: 'Datos Bancarios', type: 'textarea' } ], api: API.providers, onFinish: renderProvidersTable },
                support: { 
                    title: "Soporte", 
                    fields: [ 
                        { name: 'subject', label: 'Asunto', required: true }, 
                        { name: 'details', label: 'Detalles del Soporte', type: 'textarea' }, 
                        { name: 'associatedToType', label: 'Asociado a Tipo', type: 'select', options: ['Cliente'], required: true }, 
                        { name: 'associatedToId', label: 'Solicitante', type: 'select', options: () => Object.entries(lookups.clients || {}).map(([id, name])=>({value:id, text:name})), required: true }, 
                        { name: 'managerId', label: 'Asignar a Usuario', type: 'select', options: () => Object.entries(lookups.users || {}).map(([id, name])=>({value:id, text:name})), required: false }, 
                        { name: 'status', label: 'Estado', type: 'select', options: ['Abierto', 'En Progreso', 'Cerrado', 'Cancelado'], required: true }, 
                    ], 
                    api: API.supports, 
                    onFinish: renderSupportsTable,
                    additionalSections: {
                        comments: {
                            label: 'Comentarios',
                            render: (item) => {
                                const comments = item.comments || [];
                                let html = `<div id="comments-list" style="max-height: 200px; overflow-y: auto;">`;
                                if (comments.length === 0) {
                                    html += `<p class="text-muted">No hay comentarios.</p>`;
                                } else {
                                    comments.forEach(comment => {
                                        html += `<div class="comment-entry"><strong>${comment.user}</strong> <small>${new Date(comment.date).toLocaleString()}</small><p>${comment.text}</p></div>`;
                                    });
                                }
                                html += `</div><hr><div class="input-group mt-3"><textarea class="form-control" id="new-comment-text" placeholder="Añadir comentario..." rows="2"></textarea><button class="btn btn-primary" type="button" id="add-comment-btn">Añadir</button></div>`;
                                return html;
                            },
                            setup: (modalElement, item, onUpdate) => {
                                getEl('add-comment-btn').onclick = async () => {
                                    const commentText = getEl('new-comment-text').value.trim();
                                    if (commentText) {
                                        const newComment = {
                                            user: currentUser.fullName,
                                            date: new Date().toISOString(),
                                            text: commentText
                                        };
                                        const updatedComments = [...(item.comments || []), newComment];
                                        await API.supports.update(item.id, { comments: updatedComments });
                                        showToast('Comentario añadido.', 'success');
                                        getEl('new-comment-text').value = '';
                                        onUpdate(item.id); 
                                    }
                                };
                            },
                        },
                        attachments: {
                            label: 'Adjuntos',
                            render: (item) => {
                                const files = item.attachedFiles || [];
                                let html = `<p class="text-muted">Archivos adjuntos (simulado):</p><ul class="list-unstyled">`;
                                if (files.length === 0) {
                                    html += `<li>No hay adjuntos.</li>`;
                                } else {
                                    files.forEach(file => {
                                        html += `<li><i class="fas fa-file-alt me-2"></i>${file}</li>`;
                                    });
                                }
                                html += `</ul><input type="file" id="new-attachment-file" class="form-control mt-2" style="display: none;"><button class="btn btn-sm btn-outline-secondary mt-2" id="upload-attachment-btn"><i class="fas fa-upload me-2"></i>Subir Nuevo Adjunto (Simulado)</button>`;
                                return html;
                            },
                            setup: (modalElement, item, onUpdate) => {
                                const uploadBtn = getEl('upload-attachment-btn');
                                const fileInput = getEl('new-attachment-file');
                                uploadBtn.onclick = () => {
                                    showToast('La subida de archivos es una funcionalidad de backend. Esto es una simulación.', 'info');
                                    fileInput.click();
                                };
                                fileInput.onchange = async (e) => {
                                    if (e.target.files.length > 0) {
                                        const fileName = e.target.files[0].name;
                                        const updatedFiles = [...(item.attachedFiles || []), fileName];
                                        await API.supports.update(item.id, { attachedFiles: updatedFiles });
                                        showToast(`Adjunto '${fileName}' subido (simulado).`, 'success');
                                        onUpdate(item.id); 
                                    }
                                };
                            }
                        },
                        sendToClient: {
                            label: 'Enviar al Cliente',
                            render: (item) => `
                                <div class="mb-3">
                                    <label for="message-subject" class="form-label">Asunto del Mensaje:</label>
                                    <input type="text" class="form-control" id="message-subject" value="Actualización Ticket #${item.id}: ${item.subject}">
                                </div>
                                <div class="mb-3">
                                    <label for="message-body" class="form-label">Cuerpo del Mensaje:</label>
                                    <textarea class="form-control" id="message-body" rows="5">Estimado cliente, su ticket #${item.id} (${item.subject}) ha sido actualizado. El estado actual es: ${item.status}. ${item.comments && item.comments.length > 0 ? 'Último comentario: ' + item.comments[item.comments.length - 1].text : ''}</textarea>
                                </div>
                                <hr>
                                <div class="d-flex justify-content-around">
                                    <button class="btn btn-lg btn-outline-success mx-2" id="send-whatsapp"><i class="fab fa-whatsapp platform-icon"></i> WhatsApp</button>
                                    <button class="btn btn-lg btn-outline-info mx-2" id="send-telegram"><i class="fab fa-telegram-plane platform-icon"></i> Telegram</button>
                                    <button class="btn btn-lg btn-outline-primary mx-2" id="send-email"><i class="fas fa-envelope platform-icon"></i> Email</button>
                                    <button class="btn btn-lg btn-outline-dark mx-2" id="send-slack"><i class="fab fa-slack platform-icon"></i> Slack</button>
                                </div>
                            `,
                            setup: (modalElement, item, onUpdate) => {
                                const send = (platform) => {
                                    const subject = encodeURIComponent(getEl('message-subject').value);
                                    const body = encodeURIComponent(getEl('message-body').value);
                                    let url = '';
                                    let recipient = null;

                                    if (item.associatedToType === 'Cliente' && item.associatedToId) {
                                        recipient = allClientsData.find(c => c.id == item.associatedToId);
                                    } else { 
                                        showToast('El destinatario no es un cliente válido.', 'danger'); return;
                                    }

                                    if (!recipient) { showToast('No se encontró el destinatario (cliente).', 'danger'); return; }

                                    if (platform === 'WhatsApp') {
                                        if (!recipient.phone) { showToast('El destinatario no tiene un número de teléfono registrado.', 'danger'); return; }
                                        url = `https://api.whatsapp.com/send?phone=${recipient.phone.replace(/\D/g, '')}&text=${subject}%0A${body}`;
                                    } else if (platform === 'Telegram') {
                                        url = `https://t.me/share/url?url=&text=${subject}%0A${body}`; 
                                        showToast('Telegram requiere que el usuario seleccione el contacto después de abrir.', 'info');
                                    } else if (platform === 'Email') {
                                        if (!recipient.email) { showToast('El destinatario no tiene un email registrado.', 'danger'); return; }
                                        url = `mailto:${recipient.email}?subject=${subject}&body=${body}`;
                                    } else if (platform === 'Slack') {
                                        showToast('Simulando envío a Slack. En una aplicación real, esto integraría con una API de Slack.', 'info');
                                        return; 
                                    }
                                    
                                    if (url) {
                                        window.open(url, '_blank');
                                        showToast(`Simulando envío a ${platform}.`, 'success');
                                    }
                                };
                                getEl('send-whatsapp').onclick = () => send('WhatsApp');
                                getEl('send-telegram').onclick = () => send('Telegram');
                                getEl('send-email').onclick = () => send('Email');
                                getEl('send-slack').onclick = () => send('Slack');
                            }
                        }
                    }
                },
                fleet: { 
                    title: "Vehículo", 
                    fields: [ 
                        { name: 'id', label: 'Matrícula', required: true }, 
                        { name: 'brand', label: 'Marca', required: true }, 
                        { name: 'model', label: 'Modelo', required: true }, 
                        { name: 'photoUrl', label: 'URL de Foto' }, 
                        { name: 'responsibleId', label: 'Responsable', type: 'select', options: () => Object.entries(lookups.users || {}).map(([id, name])=>({value:id, text:name})) }, 
                        { name: 'status', label: 'Estado', type: 'select', options: ['Operativo', 'Mantenimiento', 'Fuera de Servicio'] } 
                    ], 
                    api: API.fleet, 
                    onFinish: renderFleetTable,
                    additionalSections: {
                        fines: { 
                            label: 'Multas',
                            render: (item) => {
                                const fines = item.fines || [];
                                let html = `<div id="fines-list" style="max-height: 200px; overflow-y: auto;">`;
                                if (fines.length === 0) {
                                    html += `<p class="text-muted">No hay multas registradas.</p>`;
                                } else {
                                    fines.forEach(fine => {
                                        html += `<div class="mb-2 border-bottom pb-2"><small class="text-muted">${new Date(fine.date).toLocaleDateString()}</small><br><strong>$${fine.amount}</strong> - ${fine.reason}</div>`;
                                    });
                                }
                                html += `</div><hr>
                                         <div class="mt-3">
                                            <h6 class="text-primary">Registrar Nueva Multa</h6>
                                            <div class="mb-2"><input type="date" class="form-control" id="new-fine-date" value="${new Date().toISOString().split('T')[0]}"></div>
                                            <div class="mb-2"><input type="number" class="form-control" id="new-fine-amount" placeholder="Monto" required></div>
                                            <div class="mb-2"><textarea class="form-control" id="new-fine-reason" placeholder="Razón de la multa"></textarea></div>
                                            <button class="btn btn-primary" type="button" id="add-fine-btn">Registrar Multa</button>
                                         </div>`;
                                return html;
                            },
                            setup: (modalElement, item, onUpdate) => {
                                getEl('add-fine-btn').onclick = async () => {
                                    const date = getEl('new-fine-date').value;
                                    const amount = parseFloat(getEl('new-fine-amount').value);
                                    const reason = getEl('new-fine-reason').value.trim();
                                    if (!date || isNaN(amount) || amount <= 0) {
                                        showToast('Fecha y monto válidos son requeridos.', 'danger');
                                        return;
                                    }
                                    const newFine = { date, amount, reason };
                                    const updatedFines = [...(item.fines || []), newFine];
                                    await API.fleet.update(item.id, { fines: updatedFines });
                                    showToast('Multa registrada correctamente.', 'success');
                                    getEl('new-fine-amount').value = '';
                                    getEl('new-fine-reason').value = '';
                                    onUpdate(item.id); 
                                };
                            }
                        },
                        repairs: { 
                            label: 'Reparaciones',
                            render: (item) => {
                                const repairs = item.repairs || [];
                                let html = `<div id="repairs-list" style="max-height: 200px; overflow-y: auto;">`;
                                if (repairs.length === 0) {
                                    html += `<p class="text-muted">No hay reparaciones registradas.</p>`;
                                } else {
                                    repairs.forEach(repair => {
                                        html += `<div class="mb-2 border-bottom pb-2"><small class="text-muted">${new Date(repair.date).toLocaleDateString()}</small><br><strong>${repair.description}</strong> - Costo: $${repair.cost} - Realizado por: ${repair.doneBy}</div>`;
                                    });
                                }
                                html += `</div><hr>
                                         <div class="mt-3">
                                            <h6 class="text-primary">Registrar Nueva Reparación</h6>
                                            <div class="mb-2"><input type="date" class="form-control" id="new-repair-date" value="${new Date().toISOString().split('T')[0]}"></div>
                                            <div class="mb-2"><input type="text" class="form-control" id="new-repair-description" placeholder="Descripción" required></div>
                                            <div class="mb-2"><input type="number" class="form-control" id="new-repair-cost" placeholder="Costo" required></div>
                                            <div class="mb-2"><input type="text" class="form-control" id="new-repair-doneBy" placeholder="Realizado por"></div>
                                            <button class="btn btn-primary" type="button" id="add-repair-btn">Registrar Reparación</button>
                                         </div>`;
                                return html;
                            },
                            setup: (modalElement, item, onUpdate) => {
                                getEl('add-repair-btn').onclick = async () => {
                                    const date = getEl('new-repair-date').value;
                                    const description = getEl('new-repair-description').value.trim(); 
                                    const cost = parseFloat(getEl('new-repair-cost').value);
                                    const doneBy = getEl('new-repair-doneBy').value.trim();
                                    if (!date || !description || isNaN(cost) || cost <= 0) {
                                        showToast('Fecha, descripción y costo válidos son requeridos.', 'danger');
                                        return;
                                    }
                                    const newRepair = { date, description, cost, doneBy };
                                    const updatedRepairs = [...(item.repairs || []), newRepair];
                                    await API.fleet.update(item.id, { repairs: updatedRepairs });
                                    showToast('Reparación registrada correctamente.', 'success');
                                    getEl('new-repair-description').value = '';
                                    getEl('new-repair-cost').value = '';
                                    getEl('new-repair-doneBy').value = '';
                                    onUpdate(item.id);
                                };
                            }
                        }
                    }
                },
                warehouse: { title: "Almacén", fields: [ { name: 'name', label: 'Nombre', required: true }, { name: 'location', label: 'Ubicación', required: true }, { name: 'responsibleId', label: 'Responsable', type: 'select', options: () => Object.entries(lookups.users || {}).map(([id, name])=>({value:id, text:name})) } ], api: API.warehouses, onFinish: async () => { await buildLookups(); renderWarehousesTable(); } },
                product: {
                    title: "Producto",
                    fields: [
                        { name: 'label', label: 'Descripción', required: true },
                        { name: 'type', label: 'Tipo' },
                        { name: 'brand', label: 'Marca' },
                        { name: 'model', label: 'Modelo' },
                        { name: 'minStock', label: 'Stock Mínimo', type: 'number', required: true }
                    ],
                    api: API.products,
                    onFinish: renderProductsTable
                },
                inventoryItem: {
                    title: "Ítem de Inventario",
                    fields: [
                        { name: 'serial', label: 'Serial/ID', required: true },
                        { name: 'productId', label: 'Producto', type: 'select', options: () => Object.entries(lookups.products || {}).map(([id, name])=>({value:id, text:name})), required: true },
                        { name: 'status', label: 'Estado', type: 'select', options: ['Almacén', 'Instalado', 'En Reparación', 'De Baja'], required: true },
                        { name: 'locationType', label: 'Ubicación Tipo', type: 'select', options: ['Warehouse', 'Client', 'Provider', 'N/A'], required: true },
                        { name: 'locationId', label: 'ID Ubicación', type: 'number' },
                        { name: 'notes', label: 'Notas', type: 'textarea' }, 
                        { name: 'installationDate', label: 'Fecha Instalación', type: 'date', isNewOnly: false } 
                    ],
                    api: API.inventoryItems,
                    onFinish: async () => { await buildLookups(); applyInventoryFilters(); }
                },
                inventoryMovement: {
                    title: "Movimiento de Inventario",
                    fields: [
                        { name: 'date', label: 'Fecha', type: 'date', required: true, value: new Date().toISOString().split('T')[0] },
                        { name: 'type', label: 'Tipo de Movimiento', type: 'select', options: ['Entrada', 'Salida', 'Transferencia'], required: true },
                        { name: 'inventoryItemId', label: 'Artículo Serial', type: 'select', options: () => Object.entries(lookups.inventoryItems || {}).map(([id, name])=>({value:id, text:name})), required: true },
                        { name: 'fromLocationType', label: 'Desde (Tipo)', type: 'select', options: ['Warehouse', 'Client', 'Provider', 'N/A'], required: true }, 
                        { name: 'fromLocationId', label: 'Desde (ID)', type: 'number' }, 
                        { name: 'toLocationType', label: 'Hacia (Tipo)', type: 'select', options: ['Warehouse', 'Client', 'Provider', 'N/A'], required: true }, 
                        { name: 'toLocationId', label: 'Hacia (ID)', type: 'number' }, 
                        { name: 'userId', label: 'Usuario Responsable', type: 'select', options: () => Object.entries(lookups.users || {}).map(([id, name])=>({value:id, text:name})), required: true, value: currentUser ? currentUser.id : '' },
                    ],
                    api: API.inventoryMovements,
                    onFinish: async () => { await buildLookups(); renderInventoryMovements(); applyInventoryFilters(); },
                    setup: (modalElement, item, isNew) => { 
                        const typeSelect = getEl('crud-type');
                        const fromTypeSelect = getEl('crud-fromLocationType');
                        const fromIdInput = getEl('crud-fromLocationId');
                        const toTypeSelect = getEl('crud-toLocationType');
                        const toIdInput = getEl('crud-toLocationId');

                        const updateLocationFields = () => {
                            const type = typeSelect.value;

                            fromTypeSelect.closest('.col-md-6').style.display = 'none';
                            fromIdInput.closest('.col-md-6').style.display = 'none';
                            toTypeSelect.closest('.col-md-6').style.display = 'none';
                            toIdInput.closest('.col-md-6').style.display = 'none';

                            if (type === 'Entrada') {
                                toTypeSelect.closest('.col-md-6').style.display = 'block';
                                toIdInput.closest('.col-md-6').style.display = 'block';
                                if (!isNew && !item.toLocationType) toTypeSelect.value = 'Warehouse';
                            } else if (type === 'Salida') {
                                fromTypeSelect.closest('.col-md-6').style.display = 'block';
                                fromIdInput.closest('.col-md-6').style.display = 'block';
                                if (!isNew && !item.fromLocationType) fromTypeSelect.value = 'Warehouse';
                            } else if (type === 'Transferencia') {
                                fromTypeSelect.closest('.col-md-6').style.display = 'block';
                                fromIdInput.closest('.col-md-6').style.display = 'block';
                                toTypeSelect.closest('.col-md-6').style.display = 'block';
                                toIdInput.closest('.col-md-6').style.display = 'block';
                            }
                        };
                        
                        typeSelect.addEventListener('change', updateLocationFields);
                        updateLocationFields(); 
                    }
                },
                // R: Renombrado a paymentReminder
                paymentReminder: { 
                    title: "Recordatorio de Pago",
                    fields: [
                        { name: 'clientId', label: 'Cliente Asociado', type: 'select', options: () => Object.entries(lookups.clients || {}).map(([id, name])=>({value:id, text:name})), required: true },
                        { name: 'dueDate', label: 'Fecha de Vencimiento', type: 'date', required: true },
                        { name: 'amount', label: 'Monto', type: 'number', step: '0.01', required: true },
                        { name: 'currency', label: 'Moneda', type: 'select', options: ['UYU', 'USD'], required: true },
                        { name: 'status', label: 'Estado', type: 'select', options: ['Pendiente', 'Pagado', 'Vencido'], required: true },
                        { name: 'description', label: 'Descripción', type: 'textarea' }
                    ],
                    api: API.paymentReminders, // R: Api cambiada
                    onFinish: async () => { await buildLookups(); renderPaymentMonitorView(); renderPaymentsTable(); } // R: Actualiza ambas vistas
                },
                contract: {
                    title: "Contrato",
                    fields: [
                        { name: 'clientId', label: 'Cliente Asociado', type: 'select', options: () => Object.entries(lookups.clients || {}).map(([id, name])=>({value:id, text:name})), required: true }, 
                        { name: 'type', label: 'Tipo de Contrato', required: true, options: ['Mantenimiento', 'Arriendo', 'Servicio', 'Soporte'] },
                        { name: 'startDate', label: 'Fecha Inicio', type: 'date', required: true },
                        { name: 'endDate', label: 'Fecha Fin', type: 'date' },
                        { name: 'status', label: 'Estado', type: 'select', options: ['Activo', 'Vencido', 'Inactivo', 'Pendiente'], required: true },
                        { name: 'monthlyFee', label: 'Costo Mensual', type: 'number', step: '0.01' },
                        { name: 'currency', label: 'Moneda', type: 'select', options: ['UYU', 'USD'] },
                        { name: 'terms', label: 'Términos y Condiciones', type: 'textarea' }
                    ],
                    api: API.contracts,
                    onFinish: renderContractsTable
                }
            };

            async function showCrudModal(type, id, prefillData = {}) {
                const config = modalConfig[type]; 
                const isNew = id === 'new'; 
                let item = isNew ? prefillData : await config.api.findById(id);
                if (!item && !isNew) { showToast('Elemento no encontrado', 'danger'); return; }

                const modalElement = getEl('detailModal');
                const modal = new bootstrap.Modal(modalElement);
                getEl('detailModalTitle').textContent = `${isNew ? 'Nuevo' : 'Editar'} ${config.title}`;
                
                let formFieldsHtml = '';
                for (const field of config.fields) {
                    if (field.isNewOnly && !isNew) continue; 

                    const value = (item && item[field.name] !== undefined) ? item[field.name] : (field.value !== undefined ? field.value : ''); 
                    const required = field.required ? 'required' : '';
                    let inputHtml = '';
                    if(field.type === 'select') {
                        const options = Array.isArray(field.options) ? field.options.map(o => (typeof o === 'object' ? o : {value:o, text:o})) : (typeof field.options === 'function' ? await field.options() : []);
                        inputHtml = `<select class="form-select" id="crud-${field.name}" ${required}><option value="">Seleccionar...</option>${options.map(o => `<option value="${o.value}" ${String(o.value) == String(value) ? 'selected' : ''}>${o.text}</option>`).join('')}</select>`;
                    } else if (field.type === 'textarea') { inputHtml = `<textarea class="form-control" id="crud-${field.name}" rows="3">${value}</textarea>`; } 
                    else { inputHtml = `<input type="${field.type || 'text'}" class="form-control" id="crud-${field.name}" value="${value}" ${required} ${field.step ? `step="${field.step}"`: ''}>`; }
                    formFieldsHtml += `<div class="col-md-6 mb-3"><label for="crud-${field.name}" class="form-label">${field.label}</label>${inputHtml}</div>`;
                }

                let formHtml = `<form id="crudForm" class="container-fluid" novalidate><div class="row">${formFieldsHtml}</div></form>`; 

                let additionalSectionsHtml = '';
                let tabsHtml = '';
                if (config.additionalSections && !isNew) { 
                    tabsHtml = `<ul class="nav nav-tabs" id="${type}CrudTabs" role="tablist">`;
                    tabsHtml += `<li class="nav-item" role="presentation"><button class="nav-link active" id="details-tab" data-bs-toggle="tab" data-bs-target="#details-pane" type="button" role="tab">Detalles</button></li>`;
                    
                    for (const sectionKey in config.additionalSections) {
                        const sectionConfig = config.additionalSections[sectionKey];
                        tabsHtml += `<li class="nav-item" role="presentation"><button class="nav-link" id="${sectionKey}-tab" data-bs-toggle="tab" data-bs-target="#${sectionKey}-pane" type="button" role="tab">${sectionConfig.label}</button></li>`;
                    }
                    tabsHtml += `</ul>`;
                    
                    additionalSectionsHtml = `<div class="tab-content mt-3" id="${type}CrudTabContent">`;
                    additionalSectionsHtml += `<div class="tab-pane fade show active" id="details-pane" role="tabpanel">${formHtml}</div>`;

                    for (const sectionKey in config.additionalSections) {
                        const sectionConfig = config.additionalSections[sectionKey];
                        additionalSectionsHtml += `<div class="tab-pane fade" id="${sectionKey}-pane" role="tabpanel">${sectionConfig.render(item)}</div>`;
                    }
                    additionalSectionsHtml += `</div>`;

                    getEl('detailModalBody').innerHTML = tabsHtml + additionalSectionsHtml;
                } else {
                    getEl('detailModalBody').innerHTML = formHtml; 
                }

                let footerHtml = !isNew ? `<button type="button" class="btn btn-danger me-auto" id="deleteBtn">Eliminar</button>` : '';
                footerHtml += `<button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button><button type="button" class="btn btn-primary" id="saveBtn">Guardar</button>`;
                getEl('detailModalFooter').innerHTML = footerHtml;

                const saveHandler = async () => {
                    const form = getEl('crudForm'); 
                    if (!form.checkValidity()) { form.reportValidity(); return; }
                    const formData = {};
                    for (const field of config.fields) { 
                        if(field.isNewOnly && !isNew) continue; 
                        const el = getEl(`crud-${field.name}`); 
                        if (el) { 
                            formData[field.name] = (function() {
                                if (field.type === 'number') {
                                    return el.value === '' ? null : parseFloat(el.value);
                                }
                                if (field.name.endsWith('Id') || (field.name === 'id' && type === 'fleet') || field.name === 'productId') { 
                                    return el.value === '' ? null : parseInt(el.value, 10);
                                }
                                if (el.type === 'checkbox') {
                                    return el.checked;
                                }
                                return el.value;
                            })();
                        }
                    }

                    if (type === 'inventoryMovement') {
                        const moveType = formData.type;
                        if (moveType === 'Entrada') {
                            if (!formData.toLocationType || !formData.toLocationId) { showToast('Ubicación de destino es obligatoria para Entrada.', 'danger'); return; }
                            formData.fromLocationType = "N/A";
                            formData.fromLocationId = null;
                        } else if (moveType === 'Salida') {
                            if (!formData.fromLocationType || !formData.fromLocationId) { showToast('Ubicación de origen es obligatoria para Salida.', 'danger'); return; }
                            formData.toLocationType = "N/A";
                            formData.toLocationId = null;
                        } else if (moveType === 'Transferencia') {
                            if (!formData.fromLocationType || !formData.fromLocationId || !formData.toLocationType || !formData.toLocationId) {
                                showToast('Origen y destino son obligatorios para Transferencia.', 'danger'); return;
                            }
                        }
                        const checkLocation = (locationType, locationId) => {
                            if (locationType === 'Warehouse') return AppDB_Live.warehouses.some(w => w.id === locationId);
                            if (locationType === 'Client') return AppDB_Live.clients.some(c => c.id === locationId);
                            if (locationType === 'Provider') return AppDB_Live.providers.some(p => p.id === locationId);
                            return true; 
                        };

                        if (formData.fromLocationType !== 'N/A' && !checkLocation(formData.fromLocationType, formData.fromLocationId)) {
                            showToast(`El ID de origen (${formData.fromLocationId}) para ${formData.fromLocationType} no es válido.`, 'danger'); return;
                        }
                        if (formData.toLocationType !== 'N/A' && !checkLocation(formData.toLocationType, formData.toLocationId)) {
                            showToast(`El ID de destino (${formData.toLocationId}) para ${formData.toLocationType} no es válido.`, 'danger'); return;
                        }
                    }

                    // R: Lógica para obtener lat/lon del cliente para paymentReminder si no vienen en el prefillData
                    if (type === 'paymentReminder') {
                        const client = allClientsData.find(c => c.id == formData.clientId);
                        if (client) {
                            formData.lat = client.lat;
                            formData.lon = client.lon;
                        } else {
                            showToast('Cliente no encontrado para asignar ubicación al recordatorio.', 'warning');
                            formData.lat = null;
                            formData.lon = null;
                        }
                    }


                    if (isNew) { 
                        if(type==='user' && !formData.password) { showToast('La contraseña es obligatoria para nuevos usuarios.', 'danger'); return; } 
                        await config.api.create(formData); showToast(`${config.title} creado con éxito.`); 
                    } else { 
                        delete formData.password; 
                        await config.api.update(id, formData); showToast(`${config.title} actualizado.`); 
                    }
                    modal.hide(); 
                    if (config.onFinish) config.onFinish();
                };

                getEl('saveBtn').onclick = saveHandler;

                if (!isNew) { getEl('deleteBtn').onclick = async () => { if (confirm(`¿Está seguro de que desea eliminar este ${config.title}?`)) { await config.api.delete(id); showToast(`${config.title} eliminado.`, 'warning'); modal.hide(); if (config.onFinish) config.onFinish(); } }; }
                
                modal.show();

                modalElement.addEventListener('shown.bs.modal', async () => {
                    if (config.setup) {
                        config.setup(modalElement, item, isNew);
                    }

                    if (config.additionalSections && !isNew) {
                        const crudTabsElement = getEl(`${type}CrudTabs`);
                        if (crudTabsElement) {
                            crudTabsElement.addEventListener('shown.bs.tab', async (e) => {
                                const targetPaneId = e.target.getAttribute('data-bs-target');
                                const sectionKey = targetPaneId.replace(/#|-pane/g, ''); 
                                if (sectionKey === 'details') return; 

                                const sectionConfig = config.additionalSections[sectionKey];
                                if (sectionConfig) {
                                    item = await config.api.findById(id); 
                                    const pane = getEl(sectionKey + '-pane');
                                    if (pane) {
                                        pane.innerHTML = sectionConfig.render(item); 
                                        if (sectionConfig.setup) {
                                            sectionConfig.setup(modalElement, item, async (itemId) => {
                                                const refreshedItem = await config.api.findById(itemId);
                                                pane.innerHTML = sectionConfig.render(refreshedItem);
                                                sectionConfig.setup(modalElement, refreshedItem, arguments.callee.caller.arguments[2]); 
                                            });
                                        }
                                    }
                                }
                            });
                            const activeTabButton = crudTabsElement.querySelector('.nav-link.active');
                            if (activeTabButton && activeTabButton.id !== 'details-tab') {
                                const initialSectionKey = activeTabButton.getAttribute('data-bs-target').replace(/#|-pane/g, '');
                                const sectionConfig = config.additionalSections[initialSectionKey];
                                if (sectionConfig && sectionConfig.setup) {
                                    sectionConfig.setup(modalElement, item, async (itemId) => {
                                        const refreshedItem = await config.api.findById(itemId);
                                        const pane = getEl(initialSectionKey + '-pane');
                                        if (pane) {
                                            pane.innerHTML = sectionConfig.render(refreshedItem);
                                            sectionConfig.setup(modalElement, refreshedItem, arguments.callee.caller.arguments[2]); 
                                        }
                                    });
                                }
                            }
                        }
                    }
                }, { once: true }); 
            }

            async function showClientDetailModal(clientId) {
                let client = await API.clients.findById(clientId); 
                if (!client) { showToast('Cliente no encontrado', 'danger'); return; }

                const modalElement = getEl('clientDetailModal');
                if (!modalElement) { console.error("clientDetailModal no encontrado"); return; }

                const modal = new bootstrap.Modal(modalElement);
                getEl('clientDetailModalTitle').textContent = `Detalles del Cliente: ${client.fullName}`;

                const renderClientPaymentsTab = async () => {
                    client = await API.clients.findById(clientId); 
                    const paymentsPane = getEl('client-payments-pane');
                    if (!paymentsPane) return;

                    paymentsPane.innerHTML = `
                        <h6 class="text-primary">Historial de Pagos</h6>
                        <div id="payment-history-list" style="max-height: 200px; overflow-y: auto;">
                            ${client.paymentHistory && client.paymentHistory.length > 0 ?
                                client.paymentHistory.map(p => `<div class="mb-2 border-bottom pb-2"><small class="text-muted">${new Date(p.date).toLocaleDateString()}</small><br><strong>$${p.amount}</strong> - ${p.description}</div>`).join('')
                                : '<p class="text-muted">No hay historial de pagos para este cliente.</p>'
                            }
                        </div>
                        <hr>
                        <div class="mt-3">
                            <h6 class="text-primary">Registrar Nuevo Pago</h6>
                            <div class="mb-2"><input type="date" class="form-control" id="new-client-payment-date" value="${new Date().toISOString().split('T')[0]}"></div>
                            <div class="mb-2"><input type="number" class="form-control" id="new-client-payment-amount" placeholder="Monto" required></div>
                            <div class="mb-2"><textarea class="form-control" id="new-client-payment-description" placeholder="Descripción del pago"></textarea></div>
                            <button class="btn btn-primary" type="button" id="add-client-payment-btn">Registrar Pago</button>
                        </div>
                    `;
                    getEl('add-client-payment-btn').onclick = addClientPaymentHandler; 
                };

                const addClientPaymentHandler = async () => {
                    const date = getEl('new-client-payment-date').value;
                    const amount = parseFloat(getEl('new-client-payment-amount').value);
                    const description = getEl('new-client-payment-description').value.trim();

                    if (!date || isNaN(amount) || amount <= 0) {
                        showToast('Fecha y monto válidos son requeridos para el pago.', 'danger');
                        return;
                    }

                    const newPayment = { date: date, amount: amount, description: description };
                    const updatedPaymentHistory = [...(client.paymentHistory || []), newPayment];
                    await API.clients.update(client.id, { paymentHistory: updatedPaymentHistory });
                    showToast('Pago registrado correctamente.', 'success');
                    getEl('new-client-payment-date').value = new Date().toISOString().split('T')[0];
                    getEl('new-client-payment-amount').value = '';
                    getEl('new-client-payment-description').value = '';
                    renderClientPaymentsTab(); 
                    // R: También refresca la vista global de pagos
                    renderPaymentsTable();
                };

                const renderClientInventoryTab = async () => {
                    client = await API.clients.findById(clientId); 
                    const clientInventoryBody = getEl('client-inventory-items-body');
                    if (!clientInventoryBody) return;
                    
                    const associatedItems = await API.inventoryItems.findAll(item => item.locationType === 'Client' && item.locationId == client.id);
                    if (associatedItems.length > 0) {
                        clientInventoryBody.innerHTML = associatedItems.map(item => {
                            const productLabel = lookups.products[item.productId] || 'N/A';
                            return `<tr data-item-id="${item.id}">
                                        <td>${item.serial}</td>
                                        <td>${productLabel}</td>
                                        <td><span class="badge badge-status-${item.status.toLowerCase().replace(/ /g, '-')}">${item.status}</span></td>
                                        <td>
                                            <button class="btn btn-sm btn-info transfer-item-btn" data-item-id="${item.id}" title="Transferir"><i class="fas fa-exchange-alt"></i></button>
                                            <button class="btn btn-sm btn-danger baja-item-btn" data-item-id="${item.id}" title="Dar de Baja"><i class="fas fa-trash"></i></button>
                                        </td>
                                    </tr>`;
                        }).join('');

                        qSA('#client-inventory-items-body .transfer-item-btn').forEach(btn => {
                            btn.onclick = async (e) => {
                                const itemId = parseInt(e.currentTarget.dataset.itemId, 10);
                                const itemToTransfer = await API.inventoryItems.findById(itemId);
                                if (!itemToTransfer) { showToast('Artículo no encontrado.', 'danger'); return; }

                                const newLocationType = prompt(`Transferir ${itemToTransfer.serial} a: (Warehouse/Provider)`);
                                if (!newLocationType || !['Warehouse', 'Provider'].includes(newLocationType)) {
                                    showToast('Tipo de ubicación no válido. Debe ser "Warehouse" o "Provider".', 'danger'); return;
                                }
                                const newLocationIdInput = prompt(`ID de ${newLocationType}:`);
                                const newLocationId = parseInt(newLocationIdInput, 10);

                                if (isNaN(newLocationId)) { showToast('ID de ubicación no válido.', 'danger'); return; }
                                
                                const validLocations = newLocationType === 'Warehouse' ? AppDB_Live.warehouses : AppDB_Live.providers;
                                if (!validLocations.find(loc => loc.id === newLocationId)) {
                                    showToast(`ID de ${newLocationType} no encontrado.`, 'danger'); return;
                                }

                                const newMovement = {
                                    date: new Date().toISOString().split('T')[0],
                                    type: "Transferencia",
                                    inventoryItemId: itemToTransfer.id,
                                    fromLocationType: itemToTransfer.locationType,
                                    fromLocationId: itemToTransfer.locationId,
                                    toLocationType: newLocationType,
                                    toLocationId: newLocationId,
                                    userId: currentUser.id
                                };
                                await API.inventoryItems.update(itemId, { locationType: newLocationType, locationId: newLocationId, status: newLocationType === 'Warehouse' ? 'Almacén' : 'En Reparación' });
                                await API.inventoryMovements.create(newMovement);
                                showToast('Artículo transferido y movimiento registrado.', 'success');
                                renderClientInventoryTab(); 
                            };
                        });

                        qSA('#client-inventory-items-body .baja-item-btn').forEach(btn => {
                            btn.onclick = async (e) => {
                                const itemId = parseInt(e.currentTarget.dataset.itemId, 10);
                                if (confirm('¿Confirma que desea dar de baja este artículo? Se registrará un movimiento de salida.')) {
                                    const itemToBaja = await API.inventoryItems.findById(itemId);
                                    if (!itemToBaja) { showToast('Artículo no encontrado.', 'danger'); return; }

                                    const newMovement = {
                                        date: new Date().toISOString().split('T')[0],
                                        type: "Salida", 
                                        inventoryItemId: itemToBaja.id,
                                        fromLocationType: itemToBaja.locationType,
                                        fromLocationId: itemToBaja.locationId,
                                        toLocationType: "N/A", 
                                        toLocationId: null,
                                        userId: currentUser.id
                                    };
                                    await API.inventoryItems.update(itemId, { status: "De Baja", locationType: "N/A", locationId: null });
                                    await API.inventoryMovements.create(newMovement);
                                    showToast('Artículo dado de baja.', 'success');
                                    renderClientInventoryTab(); 
                                }
                            };
                        });

                    } else {
                        clientInventoryBody.innerHTML = `<tr><td colspan="4" class="text-center text-muted">No hay ítems de inventario asociados a este cliente.</td></tr>`;
                    }

                    const assignItemSelect = getEl('assign-item-select');
                    if (assignItemSelect) {
                        const warehouseItems = await API.inventoryItems.findAll(item => item.locationType === 'Warehouse' && item.status === 'Almacén'); 
                        assignItemSelect.innerHTML = '<option value="">Seleccionar ítem en Almacén...</option>' + 
                                                   warehouseItems.map(item => `<option value="${item.id}">${item.serial} (${lookups.products[item.productId] || 'N/A'})</option>`).join('');
                        
                        getEl('assign-item-btn').onclick = async () => {
                            const selectedItemId = parseInt(assignItemSelect.value, 10);
                            if (isNaN(selectedItemId)) { showToast('Seleccione un artículo válido.', 'danger'); return; }

                            const itemToAssign = await API.inventoryItems.findById(selectedItemId);
                            if (!itemToAssign || itemToAssign.locationType !== 'Warehouse' || itemToAssign.status !== 'Almacén') { 
                                showToast('Artículo no disponible o no está en almacén.', 'danger'); return; 
                            }

                            const newMovement = {
                                date: new Date().toISOString().split('T')[0],
                                type: "Salida", 
                                inventoryItemId: itemToAssign.id,
                                fromLocationType: "Warehouse",
                                fromLocationId: itemToAssign.locationId,
                                toLocationType: "Client",
                                toLocationId: client.id,
                                userId: currentUser.id
                            };
                            await API.inventoryItems.update(selectedItemId, { 
                                status: "Instalado", 
                                locationType: "Client", 
                                locationId: client.id,
                                installationDate: new Date().toISOString().split('T')[0]
                            });
                            await API.inventoryMovements.create(newMovement);
                            showToast('Artículo asignado y movimiento registrado.', 'success');
                            renderClientInventoryTab(); 
                        };
                    }

                    const createGenericItemProductSelect = getEl('create-generic-item-product-select');
                    const createGenericItemSerialInput = getEl('create-generic-item-serial');
                    const createGenericItemBtn = getEl('create-generic-item-btn');

                    if (createGenericItemProductSelect && createGenericItemBtn) {
                        const productsForSelect = await API.products.findAll();
                        createGenericItemProductSelect.innerHTML = '<option value="">Seleccionar Producto...</option>' +
                            productsForSelect.map(p => `<option value="${p.id}">${p.label}</option>`).join('');

                        createGenericItemBtn.onclick = async () => {
                            const productId = parseInt(createGenericItemProductSelect.value, 10);
                            const serial = createGenericItemSerialInput.value.trim();

                            if (isNaN(productId) || !productId) {
                                showToast('Seleccione un producto para el nuevo ítem.', 'danger');
                                return;
                            }
                            if (!serial) {
                                showToast('El serial es obligatorio para un nuevo ítem.', 'danger');
                                return;
                            }

                            const newInventoryItemData = {
                                productId: productId,
                                serial: serial,
                                status: "Instalado",
                                locationType: "Client",
                                locationId: client.id,
                                installationDate: new Date().toISOString().split('T')[0]
                            };

                            try {
                                const newInventoryItem = await API.inventoryItems.create(newInventoryItemData);
                                showToast(`Ítem '${newInventoryItem.serial}' creado y asignado.`, 'success');

                                const newMovement = {
                                    date: new Date().toISOString().split('T')[0],
                                    type: "Entrada",
                                    inventoryItemId: newInventoryItem.id,
                                    fromLocationType: "N/A", 
                                    fromLocationId: null,
                                    toLocationType: "Client",
                                    toLocationId: client.id,
                                    userId: currentUser.id
                                };
                                await API.inventoryMovements.create(newMovement);

                                renderClientInventoryTab(); 
                                createGenericItemProductSelect.value = ''; 
                                createGenericItemSerialInput.value = '';
                            } catch (error) {
                                showToast(`Error al crear y asignar ítem: ${error.message}`, 'danger');
                            }
                        };
                    }
                };

                const renderClientContractsTab = async () => {
                    client = await API.clients.findById(clientId); 
                    const clientContractsBody = getEl('client-contracts-body');
                    if (!clientContractsBody) return;

                    const associatedContracts = await API.contracts.findAll(c => c.clientId == client.id);
                    if (associatedContracts.length > 0) {
                        clientContractsBody.innerHTML = associatedContracts.map(contract => `
                            <tr data-id="${contract.id}" data-type="contract">
                                <td>${contract.id}</td>
                                <td>${contract.type}</td>
                                <td>${contract.startDate}</td>
                                <td>${contract.endDate || 'N/A'}</td>
                                <td><span class="badge badge-status-${contract.status.toLowerCase()}">${contract.status}</span></td>
                                <td>$${contract.monthlyFee || 0} ${contract.currency || ''}</td>
                            </tr>
                        `).join('');
                        qSA('#client-contracts-body tr[data-id]').forEach(row => {
                            row.onclick = () => showCrudModal('contract', row.dataset.id);
                        });
                    } else {
                        clientContractsBody.innerHTML = `<tr><td colspan="6" class="text-center text-muted">No hay contratos asociados a este cliente.</td></tr>`;
                    }
                    getEl('btn-new-contract-for-client')?.addEventListener('click', newContractForClientHandler);
                };

                const newContractForClientHandler = () => {
                    showCrudModal('contract', 'new', { clientId: client.id }); 
                };

                const renderClientSupportsTab = async () => {
                    client = await API.clients.findById(clientId); 
                    const clientSupportsBody = getEl('client-supports-body');
                    if (clientSupportsBody) {
                        const associatedSupports = await API.supports.findAll(s => s.associatedToType === 'Cliente' && s.associatedToId == client.id);
                        if (associatedSupports.length > 0) {
                            clientSupportsBody.innerHTML = associatedSupports.map(support => `
                                <tr data-id="${support.id}" data-type="support">
                                    <td>${support.id}</td>
                                    <td>${support.subject}</td>
                                    <td><span class="badge badge-status-${support.status.toLowerCase().replace(/ /g, '-')}">${support.status}</span></td>
                                    <td>${lookups.users[support.managerId] || 'N/A'}</td>
                                    <td><button class="btn btn-sm btn-outline-info view-support-btn" data-support-id="${support.id}"><i class="fas fa-eye"></i></button></td>
                                </tr>
                            `).join('');
                            qSA('#client-supports-body .view-support-btn').forEach(btn => {
                                btn.onclick = (e) => showCrudModal('support', e.currentTarget.dataset.supportId);
                            });
                        } else {
                            clientSupportsBody.innerHTML = `<tr><td colspan="5" class="text-center text-muted">No hay soportes asociados a este cliente.</td></tr>`;
                        }
                    }
                };

                let modalBodyHtml = `
                    <div class="row h-100">
                        <div class="col-md-6 d-flex flex-column h-100">
                            <h5 class="text-primary mb-3">Ubicación del Cliente</h5>
                            <div id="modalClientMap" class="modal-map flex-grow-1"></div>
                            <small class="text-muted mt-2">Arrastre el marcador para actualizar la ubicación.</small>
                            <input type="hidden" id="client-modal-lat" value="${client.lat || ''}">
                            <input type="hidden" id="client-modal-lon" value="${client.lon || ''}">

                            <div class="text-center mb-3 mt-4">
                                <img id="client-facade-photo" src="${client.facadePhotoUrl || 'https://via.placeholder.com/400x300?text=No+Photo'}" alt="Fachada del Cliente" class="img-fluid rounded" style="width: 100%; height: auto; object-fit: cover; cursor: zoom-in;">
                            </div>
                            <input type="file" id="client-facade-photo-upload" class="d-none" accept="image/*">
                            <button class="btn btn-sm btn-outline-secondary mt-2" id="btn-upload-facade-photo"><i class="fas fa-camera me-2"></i>Subir/Cambiar Foto</button>

                        </div>
                        <div class="col-md-6 d-flex flex-column h-100">
                            <h5 class="text-primary mb-3">Información del Cliente</h5>
                            <ul class="nav nav-tabs mt-0" id="clientTabs" role="tablist">
                                <li class="nav-item" role="presentation">
                                    <button class="nav-link active" id="client-general-tab" data-bs-toggle="tab" data-bs-target="#client-general-pane" type="button" role="tab" aria-controls="client-general-pane" aria-selected="true">General</button>
                                </li>
                                <li class="nav-item" role="presentation">
                                    <button class="nav-link" id="client-payments-tab" data-bs-toggle="tab" data-bs-target="#client-payments-pane" type="button" role="tab" aria-controls="client-payments-pane" aria-selected="false">Historial de Pago</button>
                                </li>
                                <li class="nav-item" role="presentation">
                                    <button class="nav-link" id="client-inventory-tab" data-bs-toggle="tab" data-bs-target="#client-inventory-pane" type="button" role="tab" aria-controls="client-inventory-pane" aria-selected="false">Inventario</button>
                                </li>
                                <li class="nav-item" role="presentation">
                                    <button class="nav-link" id="client-supports-tab" data-bs-toggle="tab" data-bs-target="#client-supports-pane" type="button" role="tab" aria-controls="client-supports-pane" aria-selected="false">Soportes</button>
                                </li>
                                <li class="nav-item" role="presentation">
                                    <button class="nav-link" id="client-contracts-tab" data-bs-toggle="tab" data-bs-target="#client-contracts-pane" type="button" role="tab" aria-controls="client-contracts-pane" aria-selected="false">Contratos</button>
                                </li>
                            </ul>
                            <div class="tab-content flex-grow-1" id="clientTabContent">
                                <div class="tab-pane fade show active" id="client-general-pane" role="tabpanel" aria-labelledby="client-general-tab">
                                    <form id="clientDetailForm">
                                        <div class="mb-2 mt-3"><strong>ID:</strong> ${client.id}</div>
                                        <div class="mb-2">
                                            <label for="client-modal-fullName" class="form-label">Nombre Completo:</label>
                                            <input type="text" class="form-control" id="client-modal-fullName" value="${client.fullName || ''}" required>
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-contractType" class="form-label">Tipo de Contrato:</label>
                                            <select class="form-select" id="client-modal-contractType">
                                                <option value="Sin Contrato" ${client.contractType === 'Sin Contrato' ? 'selected' : ''}>Sin Contrato</option>
                                                <option value="Mantenimiento" ${client.contractType === 'Mantenimiento' ? 'selected' : ''}>Mantenimiento</option>
                                                <option value="Arriendo" ${client.contractType === 'Arriendo' ? 'selected' : ''}>Arriendo</option>
                                            </select>
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-address" class="form-label">Dirección:</label>
                                            <input type="text" class="form-control" id="client-modal-address" value="${client.address || ''}">
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-phone" class="form-label">Teléfono:</label>
                                            <input type="tel" class="form-control" id="client-modal-phone" value="${client.phone || ''}">
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-email" class="form-label">Email:</label>
                                            <input type="email" class="form-control" id="client-modal-email" value="${client.email || ''}">
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-monthlyAmount" class="form-label">Monto Mensual:</label>
                                            <input type="number" class="form-control d-inline-block" style="width: calc(100% - 70px);" id="client-modal-monthlyAmount" value="${client.monthlyAmount || 0}" step="0.01">
                                            <select class="form-select d-inline-block" style="width: 60px;" id="client-modal-monthlyCurrency">
                                                <option value="UYU" ${client.monthlyCurrency === 'UYU' ? 'selected' : ''}>UYU</option>
                                                <option value="USD" ${client.monthlyCurrency === 'USD' ? 'selected' : ''}>USD</option>
                                            </select>
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-status" class="form-label">Estado:</label>
                                            <select class="form-select" id="client-modal-status">
                                                <option value="Activo" ${client.status === 'Activo' ? 'selected' : ''}>Activo</option>
                                                <option value="Inactivo" ${client.status === 'Inactivo' ? 'selected' : ''}>Inactivo</option>
                                            </select>
                                        </div>
                                        <div class="mb-2">
                                            <label for="client-modal-notes" class="form-label">Notas:</label>
                                            <textarea class="form-control" id="client-modal-notes" rows="3">${client.notes || ''}</textarea>
                                        </div>
                                    </form>
                                </div>
                                <div class="tab-pane fade" id="client-payments-pane" role="tabpanel" aria-labelledby="client-payments-tab">
                                    <!-- Contenido de pagos se carga con renderClientPaymentsTab -->
                                </div>
                                <div class="tab-pane fade" id="client-inventory-pane" role="tabpanel" aria-labelledby="client-inventory-tab">
                                    <!-- Contenido de inventario se carga con renderClientInventoryTab -->
                                </div>
                                <div class="tab-pane fade" id="client-supports-pane" role="tabpanel" aria-labelledby="client-supports-tab">
                                    <!-- Contenido de soportes se carga con renderClientSupportsTab -->
                                </div>
                                <div class="tab-pane fade" id="client-contracts-pane" role="tabpanel" aria-labelledby="client-contracts-tab">
                                    <!-- Contenido de contratos se carga con renderClientContractsTab -->
                                </div>
                            </div>
                        </div>
                    </div>
                `;
                getEl('clientDetailModalBody').innerHTML = modalBodyHtml;

                getEl('clientDetailModalFooter').innerHTML = `
                    <button type="button" class="btn btn-danger me-auto" id="deleteClientBtn">Eliminar Cliente</button>
                    <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Cerrar</button>
                    <button type="button" class="btn btn-primary" id="saveClientDetailBtn">Guardar Cambios</button>
                `;

                modalElement.addEventListener('shown.bs.modal', async () => { 
                    const mapContainer = getEl('modalClientMap');
                    if (!mapContainer) return;
                    if (clientMapInstance) { clientMapInstance.remove(); clientMapInstance = null; } 

                    const initialLat = client.lat || -34.905;
                    const initialLon = client.lon || -56.16;

                    clientMapInstance = L.map('modalClientMap').setView([initialLat, initialLon], client.lat && client.lon ? 16 : 13);
                    L.tileLayer('https://tiles.stadiamaps.com/tiles/alidade_smooth_dark/{z}/{x}/{y}{r}.png', { attribution: '© OpenStreetMap & Stadia Maps' }).addTo(clientMapInstance);
                    
                    const color = getDotColor(client.status, client.contractType);
                    const icon = L.divIcon({ 
                        className: 'custom-dot-icon', 
                        html: `<div style="background-color: ${color};"></div>`, 
                        iconSize: [24, 24] 
                    });
                    const modalMarker = L.marker([initialLat, initialLon], { icon, draggable: true }).addTo(clientMapInstance);

                    modalMarker.on('dragend', (event) => {
                        const newCoords = event.target.getLatLng();
                        getEl('client-modal-lat').value = newCoords.lat;
                        getEl('client-modal-lon').value = newCoords.lng;
                        showToast('Nueva ubicación marcada en el mapa. Presione Guardar Cambios para aplicar.', 'info');
                    });
                    clientMapInstance.invalidateSize(); 

                    const facadePhotoUploadBtn = getEl('btn-upload-facade-photo');
                    if (facadePhotoUploadBtn) {
                        facadePhotoUploadBtn.onclick = () => getEl('client-facade-photo-upload').click();
                        getEl('client-facade-photo-upload').onchange = async (e) => {
                            if (e.target.files && e.target.files[0]) {
                                const file = e.target.files[0];
                                const reader = new FileReader();
                                reader.onload = async (e) => {
                                    const newPhotoUrl = e.target.result; 
                                    await API.clients.update(client.id, { facadePhotoUrl: newPhotoUrl });
                                    getEl('client-facade-photo').src = newPhotoUrl;
                                    showToast('Foto de fachada actualizada (simulado).', 'success');
                                };
                                reader.readAsDataURL(file);
                            }
                        };
                    }
                    getEl('client-facade-photo').onclick = () => {
                        const photoViewerModal = new bootstrap.Modal(getEl('photoViewerModal'));
                        getEl('photoViewerModalImage').src = getEl('client-facade-photo').src;
                        photoViewerModal.show();
                    };

                    const clientTabsElement = getEl('clientTabs');
                    if (clientTabsElement) {
                        clientTabsElement.addEventListener('shown.bs.tab', async (e) => {
                            const targetPaneId = e.target.getAttribute('data-bs-target');
                            switch (targetPaneId) {
                                case '#client-payments-pane': await renderClientPaymentsTab(); break;
                                case '#client-inventory-pane': await renderClientInventoryTab(); break;
                                case '#client-supports-pane': await renderClientSupportsTab(); break;
                                case '#client-contracts-pane': await renderClientContractsTab(); break;
                            }
                        });
                        const activeTabButton = clientTabsElement.querySelector('.nav-link.active');
                        if (activeTabButton && activeTabButton.id !== 'client-general-tab') {
                            const initialTargetPaneId = activeTabButton.getAttribute('data-bs-target');
                            switch (initialTargetPaneId) {
                                case '#client-payments-pane': await renderClientPaymentsTab(); break;
                                case '#client-inventory-pane': await renderClientInventoryTab(); break;
                                case '#client-supports-pane': await renderClientSupportsTab(); break;
                                case '#client-contracts-pane': await renderClientContractsTab(); break;
                            }
                        }
                    }

                }, { once: true });

                getEl('saveClientDetailBtn').onclick = async () => {
                    const updatedClient = {
                        ...client, 
                        fullName: getEl('client-modal-fullName').value,
                        contractType: getEl('client-modal-contractType').value,
                        address: getEl('client-modal-address').value,
                        phone: getEl('client-modal-phone').value,
                        email: getEl('client-modal-email').value,
                        status: getEl('client-modal-status').value,
                        monthlyAmount: parseFloat(getEl('client-modal-monthlyAmount').value), 
                        monthlyCurrency: getEl('client-modal-monthlyCurrency').value, 
                        notes: getEl('client-modal-notes').value,
                        lat: parseFloat(getEl('client-modal-lat').value),
                        lon: parseFloat(getEl('client-modal-lon').value),
                        facadePhotoUrl: getEl('client-facade-photo').src 
                    };
                    try {
                        await API.clients.update(client.id, updatedClient);
                        showToast('Cliente actualizado correctamente.', 'success'); 
                        modal.hide();
                        await buildLookups(); 
                        renderClientsTable(); 
                        if (currentView === 'geolocation') renderGeolocationMap(); 
                        renderPaymentsTable(); // R: Actualiza también la tabla global de pagos
                    } catch (error) {
                        showToast(`Error al actualizar cliente: ${error.message}`, 'danger');
                    }
                };

                getEl('deleteClientBtn').onclick = async () => {
                    if (confirm(`¿Está seguro de que desea eliminar al cliente ${client.fullName}? Esta acción es irreversible.`)) {
                        try {
                            await API.clients.delete(client.id);
                            showToast('Cliente eliminado.', 'warning');
                            modal.hide();
                            await buildLookups();
                            renderClientsTable();
                            if (currentView === 'geolocation') renderGeolocationMap();
                            renderPaymentsTable(); // R: Actualiza también la tabla global de pagos
                        } catch (error) {
                            showToast(`Error al eliminar cliente: ${error.message}`, 'danger');
                        }
                    }
                };

                modal.show();
            }

            function injectModals() { 
                getEl('modals-container').innerHTML = `
                    <div class="modal fade" id="detailModal" tabindex="-1">
                        <div class="modal-dialog modal-lg modal-dialog-centered modal-dialog-scrollable">
                            <div class="modal-content">
                                <div class="modal-header">
                                    <h5 class="modal-title" id="detailModalTitle"></h5>
                                    <button class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
                                </div>
                                <div class="modal-body" id="detailModalBody"></div>
                                <div class="modal-footer" id="detailModalFooter"></div>
                            </div>
                        </div>
                    </div>
                    <div class="modal fade" id="clientDetailModal" tabindex="-1">
                        <div class="modal-dialog modal-xl modal-dialog-centered modal-dialog-scrollable">
                            <div class="modal-content">
                                <div class="modal-header">
                                    <h5 class="modal-title" id="clientDetailModalTitle"></h5>
                                    <button class="btn-close btn-close-white" data-bs-dismiss="modal"></button>
                                </div>
                                <div class="modal-body" id="clientDetailModalBody"></div>
                                <div class="modal-footer" id="clientDetailModalFooter"></div>
                            </div>
                        </div>
                    </div>
                    <div class="modal fade" id="photoViewerModal" tabindex="-1">
                        <div class="modal-dialog modal-dialog-centered modal-lg">
                            <div class="modal-content bg-transparent border-0 shadow-none">
                                <div class="modal-body text-center">
                                    <img id="photoViewerModalImage" src="" class="img-fluid" style="max-height: 80vh; max-width: 100%; object-fit: contain;">
                                    <button type="button" class="btn-close btn-close-white position-absolute" style="top: 15px; right: 15px; background-color: rgba(0,0,0,0.5);" data-bs-dismiss="modal"></button>
                                </div>
                            </div>
                        </div>
                    </div>
                `; 
            }
            
            let universalSearchTimeout = null;
            async function handleMapClientSearch() { 
                const searchTerm = getEl('map-client-search-input').value.toLowerCase();
                const resultsContainer = getEl('map-client-search-results');
                resultsContainer.innerHTML = ''; 

                if (searchTerm.length < 2) {
                    return;
                }

                let searchResults = [];

                const clients = allClientsData.filter(c => 
                    (c.fullName && c.fullName.toLowerCase().includes(searchTerm)) ||
                    (c.address && c.address.toLowerCase().includes(searchTerm)) ||
                    (c.phone && c.phone.includes(searchTerm)) ||
                    (c.email && c.email.toLowerCase().includes(searchTerm))
                );
                clients.forEach(c => searchResults.push({ type: 'Cliente', name: c.fullName, id: c.id, lat: c.lat, lon: c.lon })); 

                if (searchResults.length > 0) {
                    resultsContainer.innerHTML = ''; 
                    searchResults.forEach(result => {
                        if (!result.lat || !result.lon) return; 
                        const item = document.createElement('button');
                        item.type = 'button';
                        item.classList.add('list-group-item', 'list-group-item-action', 'bg-dark', 'text-light', 'border-secondary');
                        item.innerHTML = `<strong>${result.type}:</strong> ${result.name} <small class="text-muted">(ID: ${result.id})</small>`;
                        item.style.cursor = 'pointer'; 
                        item.onclick = () => {
                            if (geolocationMapInstance) { 
                                geolocationMapInstance.flyTo([result.lat, result.lon], 17); 
                                geolocationMapInstance.eachLayer(layer => { 
                                    if (layer instanceof L.Marker && layer.getLatLng().lat === result.lat && layer.getLatLng().lng === result.lon) {
                                        layer.openPopup();
                                    }
                                });
                            }
                            getEl('map-client-search-container').classList.add('d-none'); 
                            resultsContainer.innerHTML = '';
                            getEl('map-client-search-input').value = '';
                        };
                        resultsContainer.appendChild(item);
                    });
                    getEl('map-client-search-container').classList.remove('d-none'); 
                } else {
                    resultsContainer.innerHTML = '<li class="list-group-item bg-dark text-muted border-secondary">No se encontraron clientes con ubicación.</li>';
                    getEl('map-client-search-container').classList.remove('d-none'); 
                }
            }

            // R: Funciones para Exportar/Importar (Implementaciones básicas en CSV)
            function exportTableToExcel(tableId, filename = 'data') {
                const table = document.getElementById(tableId);
                if (!table) {
                    showToast(`Tabla con ID '${tableId}' no encontrada para exportar.`, 'danger');
                    return;
                }

                let csv = [];
                const headers = Array.from(table.querySelectorAll('thead th')).map(th => th.textContent.trim());
                csv.push(headers.join(','));

                table.querySelectorAll('tbody tr').forEach(row => {
                    const rowData = Array.from(row.querySelectorAll('td')).map(td => {
                        let text = td.textContent.trim();
                        text = text.replace(/"/g, '""'); 
                        if (text.includes(',')) {
                            text = `"${text}"`;
                        }
                        return text;
                    });
                    csv.push(rowData.join(','));
                });

                const csvFile = new Blob([csv.join('\n')], { type: 'text/csv;charset=utf-8;' });
                const downloadLink = document.createElement('a');
                downloadLink.href = URL.createObjectURL(csvFile);
                downloadLink.download = `${filename}_${new Date().toISOString().split('T')[0]}.csv`;
                document.body.appendChild(downloadLink);
                downloadLink.click();
                document.body.removeChild(downloadLink);
                showToast(`Datos de ${filename} exportados a CSV.`, 'success');
            }

            async function importFromExcel(file, entityName, callback) {
                if (!file) {
                    showToast('Ningún archivo seleccionado para importar.', 'danger');
                    return;
                }
                if (!API[entityName] || !API[entityName].create) {
                    showToast(`Importación no soportada para la entidad '${entityName}'.`, 'danger');
                    return;
                }

                const reader = new FileReader();
                reader.onload = async (e) => {
                    const text = e.target.result;
                    const lines = text.split('\n').filter(line => line.trim() !== '');
                    if (lines.length <= 1) {
                        showToast('El archivo CSV está vacío o solo contiene encabezados.', 'warning');
                        return;
                    }

                    const headers = lines[0].split(',').map(h => h.trim());
                    const dataToImport = [];

                    for (let i = 1; i < lines.length; i++) {
                        const values = lines[i].split(','); 
                        let item = {};
                        headers.forEach((header, index) => {
                            let value = values[index] ? values[index].trim().replace(/^"|"$/g, '').replace(/""/g, '"') : ''; 
                            
                            // R: Conversión de tipos básica (mejorar según necesidad, mapear headers a nombres de campo de la DB)
                            if (header.toLowerCase().includes('id') && header.toLowerCase() !== 'productid' && header.toLowerCase() !== 'serial') { 
                                item[header] = parseInt(value, 10);
                            } else if (['amount', 'cost', 'minstock', 'monthlyamount', 'monthlyfee'].includes(header.toLowerCase())) {
                                item[header] = parseFloat(value);
                            } else if (['hascredit'].includes(header.toLowerCase())) {
                                item[header] = value.toLowerCase() === 'true' || value.toLowerCase() === 'sí';
                            } else {
                                item[header] = value;
                            }
                        });
                        dataToImport.push(item);
                    }

                    let importedCount = 0;
                    let failedCount = 0;

                    for (const item of dataToImport) {
                        try {
                            const { id, ...dataWithoutId } = item; 
                            await API[entityName].create(dataWithoutId); 
                            importedCount++;
                        } catch (error) {
                            console.error(`Error al importar item para ${entityName}:`, item, error);
                            failedCount++;
                        }
                    }
                    showToast(`Importación completada para ${entityName}: ${importedCount} items añadidos, ${failedCount} fallidos.`, importedCount > 0 ? 'success' : 'danger');
                    if (callback) callback();
                };
                reader.readAsText(file);
            }


            function setupEventListeners() {
                getEl('loginForm').addEventListener('submit', async (e) => {
                    e.preventDefault(); 
                    const loginButton = getEl('loginButton'); 
                    toggleLoading(loginButton, true);
                    try {
                        const user = await API.login(getEl('loginUsername').value, getEl('loginPassword').value, getEl('loginRole').value);
                        currentUser = user; 
                        await buildLookups(); 
                        
                        buildNavbar(user.role); 

                        getEl('login-screen').classList.add('hidden');
                        getEl('main-app').style.display = 'block'; 
                        
                        if (getEl('navbarUsernameDisplay')) {
                            getEl('navbarUsernameDisplay').textContent = user.fullName;
                        }
                        await renderNotifications(); 
                        displayView('dashboard'); 
                    } catch (error) { 
                        showToast(error.message, 'danger'); 
                    } finally { 
                        toggleLoading(loginButton, false); 
                    }
                });
                getEl('resetDataBtn').addEventListener('click', () => { if(confirm('¿Seguro que quieres borrar todos los datos guardados y volver a los datos de ejemplo? Esto es irreversible.')) { localStorage.removeItem(DB_KEY); window.location.reload(); } });
                getEl('logoutBtnNav')?.addEventListener('click', () => window.location.reload());
                
                const handleAppClick = (e) => {
                    const row = e.target.closest('tr[data-id]');
                    if (row) { 
                        const type = row.dataset.type; 
                        const id = row.dataset.id; 
                        if (type === 'client') { 
                            showClientDetailModal(id);
                        } else if (modalConfig[type]) {
                            showCrudModal(type, id); 
                        }
                    }
                    const card = e.target.closest('.kpi-card[data-view]'); if (card) displayView(card.dataset.view);
                    const navLink = e.target.closest('a[data-view]'); if(navLink) { e.preventDefault(); displayView(navLink.dataset.view); }
                };
                getEl('main-app').addEventListener('click', handleAppClick); 
                
                getEl('btn-new-client').addEventListener('click', () => showCrudModal('client', 'new')); 
                getEl('btn-new-user').addEventListener('click', () => showCrudModal('user', 'new'));
                getEl('btn-new-provider').addEventListener('click', () => showCrudModal('provider', 'new'));
                getEl('btn-new-support').addEventListener('click', () => showCrudModal('support', 'new'));
                getEl('btn-new-fleet').addEventListener('click', () => showCrudModal('fleet', 'new'));
                getEl('btn-new-warehouse').addEventListener('click', () => showCrudModal('warehouse', 'new'));
                getEl('btn-new-contract')?.addEventListener('click', () => showCrudModal('contract', 'new'));

                getEl('btn-new-product')?.addEventListener('click', () => showCrudModal('product', 'new'));
                getEl('btn-new-inventory-item')?.addEventListener('click', () => showCrudModal('inventoryItem', 'new'));
                getEl('btn-new-movement')?.addEventListener('click', () => showCrudModal('inventoryMovement', 'new'));
                // R: Botón de agregar recordatorio de pago, si se quiere habilitar
                // getEl('btn-add-monitor-device')?.addEventListener('click', () => showCrudModal('paymentReminder', 'new'));


                getEl('inventory-search')?.addEventListener('input', debounce(() => { inventoryCurrentPage = 1; applyInventoryFilters(); }, 300));
                getEl('products-search')?.addEventListener('input', debounce(() => renderProductsTable(), 300)); 
                getEl('movements-search')?.addEventListener('input', debounce(() => renderInventoryMovements(), 300)); 

                const mapClientSearchContainer = getEl('map-client-search-container');
                const mapClientSearchInput = getEl('map-client-search-input');
                const mapClientSearchResults = getEl('map-client-search-results');

                document.addEventListener('keydown', async (e) => { 
                    if (getEl('main-app').style.display === 'block') {
                        if (e.key === 'F3' && (currentView === 'geolocation' || currentView === 'monitor')) { 
                            e.preventDefault();
                            mapClientSearchContainer.classList.remove('d-none');
                            mapClientSearchInput.focus();
                            mapClientSearchInput.value = ''; 
                            mapClientSearchResults.innerHTML = '';
                        }
                        if (e.key === 'Escape' && !mapClientSearchContainer.classList.contains('d-none')) {
                            e.preventDefault();
                            mapClientSearchContainer.classList.add('d-none');
                            mapClientSearchResults.innerHTML = '';
                            mapClientSearchInput.value = '';
                        }
                        if (!(currentView === 'geolocation' || currentView === 'monitor') && !mapClientSearchContainer.classList.contains('d-none')) {
                             mapClientSearchContainer.classList.add('d-none');
                             mapClientSearchResults.innerHTML = '';
                             mapClientSearchInput.value = '';
                        }
                        if (e.key === 'Enter' && !mapClientSearchContainer.classList.contains('d-none')) {
                            const firstResult = qS('#map-client-search-results button.list-group-item');
                            if (firstResult) {
                                firstResult.click();
                            }
                        }
                    }
                });

                mapClientSearchInput.addEventListener('input', debounce(() => {
                    handleMapClientSearch();
                }, 300)); 
                
                qSA('[id^="btn-export-"]').forEach(button => {
                    button.addEventListener('click', (e) => {
                        const tableIdPrefix = e.target.id.replace('btn-export-', ''); 
                        let actualTableId;
                        let filename;

                        const idMap = {
                            'clients': 'clientsTable',
                            'users': 'usersTable',
                            'providers': 'providersTable',
                            'supports': 'supportsTable',
                            'fleet': 'fleetTable',
                            'warehouses': 'warehousesTable',
                            'products': 'productsTable',
                            'inventoryItems': 'inventoryItemsTable',
                            'inventoryMovements': 'inventoryMovementsTable',
                            'paymentReminders': 'paymentsTable', // R: Ahora apunta a paymentsTable
                            'contracts': 'contractsTable', 
                            'payments': 'paymentsTable' 
                        };

                        actualTableId = idMap[tableIdPrefix];
                        filename = tableIdPrefix;

                        if (getEl(actualTableId)) {
                            exportTableToExcel(actualTableId, filename);
                        } else {
                            console.warn(`No se encontró la tabla para exportar con ID: ${actualTableId}`);
                        }
                    });
                });

                qSA('[id^="btn-import-"]').forEach(button => {
                    const entityName = button.id.replace('btn-import-', '');
                    const fileInputId = `file-input-${entityName}`;
                    const fileInput = getEl(fileInputId);

                    if (fileInput) {
                        button.addEventListener('click', () => fileInput.click());
                        fileInput.addEventListener('change', (e) => {
                            if (e.target.files.length > 0) {
                                const file = e.target.files[0];
                                importFromExcel(file, entityName, async () => {
                                    await buildLookups(); 
                                    if (renderFunctions[entityName]) {
                                        renderFunctions[entityName]();
                                    } else if (entityName === 'inventoryItems') {
                                        applyInventoryFilters(); 
                                    } else if (entityName === 'inventoryMovements') {
                                        renderInventoryMovements();
                                    } else if (entityName === 'paymentReminders') { // R: Cambiado a paymentReminders
                                        renderPaymentMonitorView(); // R: Renderiza el monitor de pagos
                                        renderPaymentsTable(); // R: Y la tabla de pagos
                                    } else if (entityName === 'contracts') { 
                                        renderContractsTable();
                                    }
                                });
                            }
                        });
                    }
                });

                getEl('monitor-filter-status')?.addEventListener('change', renderPaymentMonitorView); // R: Ahora llama a renderPaymentMonitorView

            }

            document.addEventListener('DOMContentLoaded', () => {
                document.documentElement.setAttribute('data-bs-theme', 'dark'); 
                injectModals();
                setupEventListeners();
            });

        } catch (err) { 
            console.error("Error fatal en la inicialización de la aplicación:", err);
            document.body.innerHTML = `<div class="vh-100 d-flex flex-column justify-content-center align-items-center text-danger"><h1><i class="fas fa-bomb fa-2x"></i></h1><h2 class="mt-3">Error Fatal en la Aplicación</h2><p class="lead">${err.message || 'Error desconocido'}</p><p>Por favor, recargue la página o contacte al soporte.</p></div>`;
        }
    })(); 
    </script>
</body>
</html>
