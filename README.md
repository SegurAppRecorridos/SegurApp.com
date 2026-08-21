<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Carnet Digital Interactivo - SegurApp Recorridos</title>
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
    <!-- html2canvas para la descarga del carnet como imagen -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <!-- Leaflet CSS y JS para el mapa interactivo de rutas -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY=" crossorigin=""/>
    <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js" integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo=" crossorigin=""></script>
    <style>
        :root {
            --bg-principal: #08080a;
            --bg-tarjeta: #ffffff;
            --bg-input: #f0f2f5;
            --acero-claro: #a6b4c9;
            --acero-oscuro: #3a4454;
            --dorado-brillante: #d4af37;
            --dorado-base: #b88628;
            --dorado-oscuro: #997a15;
            --glow-dorado: rgba(255, 255, 255, 0.4);
            --verde-verificado: #00aa55;
            --glow-verde: rgba(0, 170, 85, 0.4);
            
            --texto-principal: #1a1a1a;
            --texto-secundario: #555555;
            --borde-sutil: rgba(212, 175, 55, 0.6);
            --fuente: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            -webkit-tap-highlight-color: transparent;
            font-family: var(--fuente);
        }

        body {
            background-image: url('https://imgs.search.brave.com/G8vLnjhvCWZGYZRYmQ47ylrIKpEtItCPSqBLRGXJL6I/rs:fit:860:0:0:0/g:ce/aHR0cHM6Ly9kZWxj/YXIuY29tLnV5L3dw/LWNvbnRlbnQvdXBs/b2Fkcy8yMDIzLzEw/L1JBSURFUi0xNy1z/Y2FsZWQuanBn');
            background-size: cover;
            background-position: center;
            background-repeat: no-repeat;
            background-attachment: fixed;
            margin: 0;
            color: var(--texto-principal);
            min-height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 20px;
            position: relative;
            overflow-x: hidden;
        }

        .background-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.3);
            z-index: -1;
        }

        body::before {
            content: '';
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: radial-gradient(circle at center, rgba(212, 175, 55, 0.08) 0%, rgba(8, 8, 10, 0.88) 80%);
            z-index: -1;
            pointer-events: none;
        }

        #toastContainer {
            position: fixed;
            top: 20px;
            right: 20px;
            z-index: 99999;
            display: flex;
            flex-direction: column;
            gap: 10px;
            max-width: 360px;
            width: 100%;
            pointer-events: none;
        }

        .custom-toast {
            background: rgba(12, 15, 22, 0.95);
            border: 2px solid var(--dorado-brillante);
            border-radius: 14px;
            padding: 14px 18px;
            color: #fff;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 20px var(--glow-dorado);
            display: flex;
            align-items: flex-start;
            gap: 12px;
            pointer-events: auto;
            transform: translateX(120%);
            transition: transform 0.4s cubic-bezier(0.175, 0.885, 0.32, 1.275), opacity 0.3s ease;
            opacity: 0;
            backdrop-filter: blur(10px);
        }

        .custom-toast.show {
            transform: translateX(0);
            opacity: 1;
        }

        .custom-toast.success {
            border-color: var(--verde-verificado);
            box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 20px var(--glow-verde);
        }

        .custom-toast.error {
            border-color: #ff4444;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 20px rgba(255, 68, 68, 0.4);
        }

        .toast-icon {
            font-size: 24px;
            color: var(--dorado-brillante);
            flex-shrink: 0;
        }

        .custom-toast.success .toast-icon {
            color: var(--verde-verificado);
        }

        .custom-toast.error .toast-icon {
            color: #ff4444;
        }

        .toast-content {
            flex: 1;
            display: flex;
            flex-direction: column;
            gap: 2px;
        }

        .toast-title {
            font-size: 0.9rem;
            font-weight: 800;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            color: #fff;
        }

        .toast-msg {
            font-size: 0.82rem;
            color: #ccc;
            line-height: 1.4;
        }

        .toast-close {
            background: none;
            border: none;
            color: #aaa;
            font-size: 18px;
            cursor: pointer;
            padding: 0;
            line-height: 1;
        }

        .toast-close:hover {
            color: #fff;
        }

        .interactive-alert-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(8px);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 100000;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
            padding: 20px;
        }

        .interactive-alert-overlay.active {
            opacity: 1;
            pointer-events: auto;
        }

        .interactive-alert-box {
            background: #0b0d12;
            border: 2px solid var(--dorado-brillante);
            border-radius: 20px;
            width: 100%;
            max-width: 400px;
            padding: 24px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.9), 0 0 30px var(--glow-dorado);
            display: flex;
            flex-direction: column;
            gap: 16px;
            text-align: center;
            transform: scale(0.8);
            transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        .interactive-alert-overlay.active .interactive-alert-box {
            transform: scale(1);
        }

        .interactive-alert-icon {
            font-size: 48px;
            color: var(--dorado-brillante);
            margin: 0 auto;
            text-shadow: 0 0 15px var(--glow-dorado);
        }

        .interactive-alert-title {
            font-size: 1.15rem;
            font-weight: 900;
            color: #fff;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .interactive-alert-msg {
            font-size: 0.92rem;
            color: var(--texto-secundario);
            line-height: 1.5;
        }

        .interactive-alert-input {
            background: var(--bg-input);
            border: 1px solid var(--borde-sutil);
            color: #1a1a1a;
            padding: 12px 14px;
            border-radius: 10px;
            font-size: 0.95rem;
            outline: none;
            width: 100%;
            text-align: center;
        }

        .interactive-alert-input:focus {
            border-color: var(--dorado-brillante);
            box-shadow: 0 0 10px var(--glow-dorado);
        }

        .interactive-alert-buttons {
            display: flex;
            gap: 10px;
            margin-top: 4px;
        }

        .interactive-alert-btn {
            flex: 1;
            padding: 12px;
            border-radius: 10px;
            font-weight: 800;
            font-size: 0.9rem;
            cursor: pointer;
            text-transform: uppercase;
            border: none;
            transition: all 0.2s ease;
        }

        .interactive-alert-btn.primary {
            background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
            color: #000;
            box-shadow: 0 0 15px var(--glow-dorado);
        }

        .interactive-alert-btn.secondary {
            background: rgba(255, 255, 255, 0.1);
            color: #fff;
            border: 1px solid var(--borde-sutil);
        }

        .user-top-bar {
            width: 100%;
            max-width: 380px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            background: rgba(18, 21, 28, 0.9);
            border: 1px solid var(--borde-sutil);
            padding: 8px 14px;
            border-radius: 30px;
            margin-bottom: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.5);
        }

        .user-info-pill {
            display: flex;
            align-items: center;
            gap: 10px;
            font-size: 0.85rem;
            color: #fff;
            font-weight: 700;
        }

        .user-avatar-mini {
            width: 32px;
            height: 32px;
            border-radius: 50%;
            border: 1.5px solid var(--dorado-brillante);
            object-fit: cover;
            background-color: #000;
        }

        .top-bar-buttons {
            display: flex;
            gap: 6px;
        }

        .btn-auth-trigger {
            background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
            color: #000;
            border: none;
            padding: 12px 12px;
            border-radius: 20px;
            font-size: 0.78rem;
            font-weight: 800;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 4px;
            text-transform: uppercase;
        }

        .btn-edit-trigger {
            background: rgba(212, 175, 55, 0.15);
            color: var(--dorado-brillante);
            border: 1px solid var(--dorado-brillante);
            padding: 6px 10px;
            border-radius: 20px;
            font-size: 0.78rem;
            font-weight: 800;
            cursor: pointer;
            display: flex;
            align-items: center;
            gap: 4px;
            text-transform: uppercase;
        }

        .hint-girar {
            font-size: 0.95rem;
            color: #ffffff;
            text-transform: uppercase;
            letter-spacing: 1.2px;
            font-weight: 900;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
            margin-bottom: 16px;
            padding: 10px 20px;
            background: linear-gradient(135deg, rgba(100, 103, 108, 0.95) 0%, rgba(10, 12, 16, 0.95) 100%);
            border: 2px solid var(--dorado-brillante);
            border-radius: 30px;
            box-shadow: 0 0 15px rgba(255, 215, 0, 0.4), inset 0 0 10px rgba(255, 215, 0, 0.1);
            animation: pulseHint 1.8s infinite ease-in-out;
            cursor: pointer;
            user-select: none;
        }

        .hint-girar .icono-girar {
            font-size: 22px;
            color: var(--dorado-brillante);
            animation: rotarIcono 2.5s infinite linear;
        }

        @keyframes pulseHint {
            0%, 100% {
                transform: scale(1);
                box-shadow: 0 0 12px rgba(255, 215, 0, 0.4), 0 0 20px rgba(212, 175, 55, 0.2);
            }
            50% {
                transform: scale(1.05);
                box-shadow: 0 0 22px rgba(255, 215, 0, 0.8), 0 0 35px rgba(212, 175, 55, 0.5);
                border-color: #ffe600;
            }
        }

        @keyframes rotarIcono {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .escena-carnet {
            width: 100%;
            max-width: 420px;
            height: 680px;
            perspective: 1200px;
            margin-bottom: 20px;
            cursor: pointer;
        }

        .carnet-inner { 
            width: 100%;
            height: 100%;
            position: relative;
            transform-style: preserve-3d;
            transition: transform 0.8s cubic-bezier(0.175, 0.885, 0.32, 1.275);
        }

        .escena-carnet.flipped .carnet-inner {
            transform: rotateY(180deg);
        }

        .carnet-cara {
            position: absolute;
            width: 100%;
            height: 100%;
            backface-visibility: hidden;
            -webkit-backface-visibility: hidden;
            background: var(--bg-tarjeta);
            border: 2.5px solid var(--dorado-brillante);
            border-radius: 20px;
            padding: 14px 18px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            overflow: hidden;
            box-shadow: 0 15px 35px rgba(0, 0, 0, 0.7), 0 0 30px rgba(255, 255, 255, 0.5);
        }

        .carnet-cara.reverso {
            transform: rotateY(180deg);
        }   

        @keyframes barridoMetalico {
            0% { left: -150%; }
            50% { left: -150%; }
            100% { left: 150%; }
        }

        @keyframes brilloPulsante {
            0%, 100% { box-shadow: 0 15px 35px rgba(0, 0, 0, 0.7), 0 0 20px rgba(255, 255, 255, 0.3); }
            50% { box-shadow: 0 15px 35px rgba(0, 0, 0, 0.7), 0 0 40px rgba(255, 255, 255, 0.6); }
        }

        .carnet-cara {
            animation: brilloPulsante 3s infinite ease-in-out;
        }

        .carnet-cara::after {
            content: '';
            position: absolute;
            top: 0;
            width: 100%;
            height: 100%;
            background: linear-gradient(
                90deg,
                rgba(255, 215, 0, 0) 0%,
                rgba(255, 215, 0, 0.02) 20%,
                rgba(212, 175, 55, 0.1) 50%,
                rgba(255, 215, 0, 0.02) 80%,
                rgba(255, 215, 0, 0) 100%
            );
            transform: skewX(-25deg);
            animation: barridoMetalico 6s infinite ease-in-out;
            pointer-events: none;
        }

        .perforacion-lanyard {
            width: 50px;
            height: 10px;
            background: #dcdcdc;
            border: 1px solid #b88628;
            border-radius: 10px;
            margin: 0 auto 4px auto;
            box-shadow: inset 0 2px 4px rgba(0,0,0,0.2);
        }

        .carnet-header {
            text-align: center;
            border-bottom: 1.5px solid rgba(251, 251, 248, 0.4);
            padding-bottom: 4px;
            margin-bottom: 2px;
            display: flex;  
            flex-direction: column;
            align-items: center;
        }

        .carnet-header img.logo {
            max-width: 50px;
            height: auto;
            margin-bottom: 2px;
        }

        .frontal .carnet-header .sub-brand, .reverso .carnet-header .sub-brand {
            font-size: 0.84rem;
            color: #997a15;
            letter-spacing: 1.5px;
            text-transform: uppercase;
            font-weight: 800;
        }

        .badge-rol {
            display: inline-block;
            background: #111111;
            color: var(--dorado-brillante);
            border: 1.5px solid var(--dorado-brillante);
            font-size: 0.84rem;
            font-weight: 900;
            padding: 3px 12px;
            border-radius: 20px;
            text-transform: uppercase;
            letter-spacing: 1px;
            margin-top: 2px;
            box-shadow: 0 0 10px var(--glow-dorado);
        }

        .carnet-body {
            display: flex;
            flex-direction: column;
            align-items: center;
            text-align: center;
        }

        .foto-marco {
            position: relative;
            margin-top: 4px;
            margin-bottom: 2px;
        }

        .foto-conductor {
            width: 170px;
            height: 170px;
            border-radius: 50%;
            object-fit: cover;
            border: 3px solid var(--dorado-brillante);
            box-shadow: 0 0 18px var(--glow-dorado);
            background-color: #f0f0f0;
        }

        .badge-verificado-icono {
            position: absolute;
            bottom: 15px;
            right: 11px;
            background: #1e501c;
            color: #00ff88;
            border-radius: 50%;
            width: 30px;
            height: 30px;
            display: flex;
            align-items: center;
            justify-content: center;
            border: 2px solid #00ff88;
            box-shadow: 0 0 15px rgba(0, 255, 136, 0.5);
            z-index: 10;
        }

        .badge-verificado-icono::after {
            content: '';
            position: absolute;
            width: 100%;
            height: 100%;
            border-radius: 50%;
            border: 2px solid #00ff88;
            opacity: 0;
            animation: pulsoVerde 2s infinite;
            box-sizing: border-box;
        }

        @keyframes pulsoVerde {
            0% { transform: scale(1); opacity: 1; }
            100% { transform: scale(1.5); opacity: 0; }
        }

        .frontal .nombre-conductor {
            font-size: 1.26rem;
            font-weight: 900;
            color: #111111;
            margin-top: 2px;
            margin-bottom: 2px;
        }

        .frontal .cargo-conductor {
            font-size: 0.88rem;
            color: #555555;
            font-weight: 600;
            margin-bottom: 8px;
        }

        .datos-grid {
            width: 100%;
            background: #f8f9fc;
            border-radius: 12px;
            padding: 8px 10px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 6px;
            text-align: left;
            border: 1px solid rgba(212, 175, 55, 0.4);
            margin-bottom: 8px;
        }

        .dato-item {
            display: flex;
            flex-direction: column;
        }

        .dato-item.full-width {
            grid-column: span 2;
        }

        .dato-label {
            font-size: 0.75rem;
            color: #666666;
            text-transform: uppercase;
            font-weight: 700;
            letter-spacing: 0.3px;
        }

        .dato-val {
            font-size: 0.90rem;
            color: #1a1a1a;
            font-weight: 800;
        }

        .dato-val.destacado {
            color: #997a15;
        }

        .qr-destacado-container {
            display: flex;
            justify-content: center;
            align-items: center;
            margin: 4px 0 8px 0;
        }

        .qr-code-lg {
            width: 120px;
            height: 120px;
            background: #ffffff;
            padding: 8px;
            border-radius: 14px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.15), 0 0 15px var(--glow-dorado);
            border: 2.5px solid var(--dorado-brillante);
        }

        .qr-code-lg img {
            width: 100%;
            height: 100%;
            display: block;
            object-fit: contain;
        }

        .carnet-footer {
            display: flex;
            align-items: center;
            justify-content: space-around;
            width: 100%;
            background: #ffffff;
            padding: 8px 12px;
            border-radius: 12px;
            border: 1px dashed rgba(212, 175, 55, 0.5);
        }

        .info-qr {
            text-align: center;
            width: 100%;
            display: flex;
            justify-content: space-around;
            align-items: center;
        }

        .info-qr .id-codigo {
            font-family: monospace;
            font-size: 0.90rem;
            color: #1a1a1a;
            font-weight: 800;
        }

        .info-qr .estado {
            font-size: 0.80rem;
            color: #008844;
            font-weight: 800;
            text-transform: uppercase;
            display: flex;
            align-items: center;
            gap: 4px;
        }

        .rating-box-reverso {
            width: 100%;
            background: #f8f9fc;
            border: 1px solid rgba(212, 175, 55, 0.4);
            border-radius: 12px;
            padding: 10px;
            margin: 10px 0;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .rating-title {
            font-size: 0.78rem;
            color: #997a15;
            text-transform: uppercase;
            font-weight: 800;
            letter-spacing: 0.5px;
            margin-bottom: 4px;
        }

        .rating-promedio {
            font-size: 0.98rem;
            font-weight: 900;
            color: #1a1a1a;
            display: flex;
            align-items: center;
            gap: 4px;
        }

        .stars-container {
            display: flex;
            gap: 6px;
            margin: 6px 0;
        }

        .star-icon {
            font-size: 24px;
            color: #cccccc;
            cursor: pointer;
            transition: all 0.2s ease;
        }

        .star-icon.active, .star-icon:hover {
            color: #d4af37;
            text-shadow: 0 0 10px rgba(212, 175, 55, 0.6);
        }

        .voto-mensaje {
            font-size: 0.78rem;
            color: #008844;
            margin-top: 2px;
            font-weight: 800;
        }

        .reverso-info {
            font-size: 0.85rem;
            color: #444444;
            line-height: 1.4;
            text-align: center;
        }

        .acciones-bar {
            width: 100%;
            max-width: 420px;
            display: flex;
            flex-direction: column;
            gap: 12px;
        }

        .btn-accion {
            flex: 1;
            background: linear-gradient(145deg, #111318 0%, #1a1d24 100%);
            border: 2px solid var(--dorado-brillante);
            color: #ffffff;
            padding: 16px 20px;
            border-radius: 16px;
            font-size: 1.05rem;
            font-weight: 900;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 10px;
            transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
            text-transform: uppercase;
            letter-spacing: 1px;
            box-shadow: 0 6px 15px rgba(0,0,0,0.6), inset 0 2px 5px rgba(255,255,255,0.05);
            position: relative;
            overflow: hidden;
            text-decoration: none;
        }

        .btn-accion:hover {
            transform: translateY(-3px);
            box-shadow: 0 10px 20px rgba(212, 175, 55, 0.4);
            background: linear-gradient(145deg, #1a1d24 0%, #242933 100%);
        }

        .btn-accion:active {
            transform: scale(0.94) translateY(2px);
            box-shadow: 0 2px 10px rgba(212, 175, 55, 0.2);
        }

        .btn-accion::after {
            content: '';
            position: absolute;
            top: 0;
            left: -100%;
            width: 50%;
            height: 100%;
            background: linear-gradient(to right, rgba(255,255,255,0) 0%, rgba(255, 215, 0, 0.3) 50%, rgba(255,255,255,0) 100%);
            transform: skewX(-25deg);
            transition: all 0.5s ease;
        }

        .btn-accion:hover::after {
            left: 200%;
            transition: all 0.7s ease;
        }

        .btn-accion.solicitar-wasap {
            background: linear-gradient(135deg, #00ff88 0%, #056232 100%);
            color: #000000;
            border: 3px solid #ffffff;
            font-size: 1.25rem;
            padding: 20px 24px;
            border-radius: 20px;
            box-shadow: 0 0 25px rgba(0, 255, 136, 0.5), inset 0 4px 6px rgba(255,255,255,0.4);
            animation: latidoMotor 1.5s infinite ease-in-out;
            width: 100%;
        }

        @keyframes latidoMotor {
            0%, 100% { 
                transform: scale(1); 
                box-shadow: 0 0 20px rgba(0, 255, 136, 0.4); 
            }
            50% { 
                transform: scale(1.03); 
                box-shadow: 0 0 35px rgba(0, 255, 136, 0.8), inset 0 2px 10px rgba(255,255,255,0.6); 
                border-color: #00ff88; 
            }
        }

        .btn-accion.quienes-somos {
            padding: 8px 5px;
            background: rgba(22, 25, 32, 0.9);
            border: 1px solid var(--dorado-brillante);
            color: var(--dorado-brillante);
            box-shadow: 0 0 10px rgba(212, 175, 55, 0.2);
        }

        .btn-accion.terminos-condiciones {
            padding: 8px 4px;
            background: rgba(22, 25, 32, 0.9);
            border: 1px solid #ff4444;
            color: #ff6666;
            box-shadow: 0 0 10px rgba(255, 68, 68, 0.2);
        }

        .modal-overlay {
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.85);
            backdrop-filter: blur(8px);
            display: flex;
            align-items: center;
            justify-content: center;
            z-index: 1000;
            opacity: 0;
            pointer-events: none;
            transition: opacity 0.3s ease;
            padding: 20px;
        }

        .modal-overlay.active {
            opacity: 1;
            pointer-events: auto;
        }

        .modal-contenido {
            background: #000000;
            border: 1px solid var(--borde-sutil);
            border-radius: 20px;
            width: 100%;
            max-width: 480px;
            max-height: 88vh;
            overflow-y: auto;
            padding: 24px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.8), 0 0 25px var(--glow-dorado);
            position: relative;
            transform: translateY(20px);
            transition: transform 0.3s ease;
        }

        .modal-overlay.active .modal-contenido {
            transform: translateY(0);
        }

        .modal-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid var(--borde-sutil);
            padding-bottom: 12px;
            margin-bottom: 16px;
        }

        .modal-titulo {
            font-size: 1.15rem;
            font-weight: 800;
            color: var(--dorado-brillante);
            text-transform: uppercase;
            letter-spacing: 1px;
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .btn-cerrar {
            background: none;
            border: none;
            color: var(--texto-secundario);
            font-size: 26px;
            cursor: pointer;
            display: flex;
            align-items: center;
            justify-content: center;
        }

        .modal-body {
            font-size: 0.92rem;
            color: #d1d7e0;
            line-height: 1.6;
            display: flex;
            flex-direction: column;
            gap: 14px;
        }

        .modal-body h4, .terminos-titulo-seccion {
            color: var(--dorado-brillante) !important;
            font-size: 1.02rem;
            margin-top: 6px;
            display: flex;
            align-items: center;
            gap: 6px;
            border-bottom: 1px solid rgba(212, 175, 55, 0.3);
            padding-bottom: 4px;
            text-shadow: 0 0 8px rgba(212, 175, 55, 0.4);
        }

        .caracteristica-box {
            background: #12151c;
            border: 1px solid var(--borde-sutil);
            border-radius: 10px;
            padding: 10px;
            margin-top: 4px;
        }

        .auth-tabs {
            display: flex;
            gap: 6px;
            margin-bottom: 16px;
            border-bottom: 1px solid var(--borde-sutil);
            padding-bottom: 10px;
        }

        .auth-tab-btn {
            flex: 1;
            background: transparent;
            border: none;
            color: #aaa;
            padding: 8px 4px;
            font-weight: 700;
            font-size: 0.78rem;
            cursor: pointer;
            border-radius: 8px;
            transition: all 0.2s ease;
            text-align: center;
        }

        .auth-tab-btn.active {
            background: var(--dorado-brillante);
            color: #000;
        }

        .form-group {
            display: flex;
            flex-direction: column;
            gap: 6px;
            text-align: left;
            margin-bottom: 10px;
        }

        .form-group label {
            font-size: 0.8rem;
            font-weight: 700;
            color: var(--dorado-brillante);
            text-transform: uppercase;
        }

        .form-group input, .form-group select, .form-group textarea {
            background: #12151c;
            border: 1px solid var(--borde-sutil);
            color: #fff;
            padding: 10px 10px;
            border-radius: 10px;
            font-size: 0.9rem;
            outline: none;
            width: 100%;
        }

        .form-group input:focus, .form-group select:focus, .form-group textarea:focus {
            border-color: var(--dorado-brillante);
            box-shadow: 0 0 8px var(--glow-dorado);
        }

        .avatar-upload-preview {
            display: flex;
            align-items: center;
            gap: 12px;
            margin-bottom: 10px;
        }

        .preview-circle {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            border: 2px solid var(--dorado-brillante);
            object-fit: cover;
            background: #111;
        }

        .btn-submit-auth {
            background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
            color: #000;
            border: none;
            padding: 12px;
            border-radius: 10px;
            font-weight: 800;
            font-size: 0.95rem;
            cursor: pointer;
            text-transform: uppercase;
            letter-spacing: 0.5px;
            margin-top: 8px;
            width: 100%;
            box-shadow: 0 0 10px var(--glow-dorado);
        }

        .link-forgot-pass {
            font-size: 0.8rem;
            color: var(--dorado-brillante);
            text-decoration: underline;
            cursor: pointer;
            text-align: right;
            display: block;
            margin-top: 4px;
        }

        .alerta-seguridad-box {
            position: relative;
            background: linear-gradient(145deg, rgba(35, 10, 10, 0.95) 0%, rgba(15, 5, 5, 0.98) 100%);
            border: 2px solid #ff3333;
            border-radius: 16px;
            padding: 16px;
            color: #ffffff;
            font-size: 0.9rem;
            box-shadow: 0 0 20px rgba(255, 51, 51, 0.35);
            margin-bottom: 6px;
        }

        .alerta-header {
            display: flex;
            align-items: center;
            justify-content: space-between;
            border-bottom: 1px solid rgba(255, 85, 85, 0.3);
            padding-bottom: 8px;
            margin-bottom: 10px;
        }

        .alerta-titulo-text {
            display: flex;
            align-items: center;
            gap: 8px;
            color: #ff5555;
            font-weight: 900;
            font-size: 0.95rem;
            letter-spacing: 0.8px;
            text-transform: uppercase;
        }

        .badge-live-seguridad {
            background: rgba(255, 51, 51, 0.2);
            color: #ff6666;
            border: 1px solid #ff4444;
            font-size: 0.68rem;
            font-weight: 800;
            padding: 2px 8px;
            border-radius: 12px;
            display: flex;
            align-items: center;
            gap: 4px;
            text-transform: uppercase;
        }

        .dot-live {
            width: 7px;
            height: 7px;
            background-color: #ff3333;
            border-radius: 50%;
            box-shadow: 0 0 8px #ff3333;
            animation: pulsoRedLive 1.2s infinite ease-in-out;
        }

        @keyframes pulsoRedLive {
            0%, 100% { opacity: 1; transform: scale(1); }
            50% { opacity: 0.3; transform: scale(1.4); }
        }

        .alerta-puntos-claves {
            display: grid;
            grid-template-columns: 1fr;
            gap: 6px;
        }

        .punto-seguridad-item {
            background: rgba(0, 0, 0, 0.6);
            border: 1px solid rgba(255, 85, 85, 0.25);
            border-radius: 8px;
            padding: 6px 10px;
            display: flex;
            align-items: center;
            gap: 8px;
            font-size: 0.82rem;
            color: #ffffff;
            font-weight: 600;
        }

        .btn-aceptar-terminos {
            width: 100%;
            background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%);
            color: #000;
            border: none;
            padding: 14px;
            border-radius: 12px;
            font-weight: 800;
            font-size: 1rem;
            cursor: pointer;
            text-transform: uppercase;
            letter-spacing: 1px;
            box-shadow: 0 0 15px var(--glow-dorado);
            margin-top: 10px;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }
    </style>
</head> 
<body>

    <div id="toastContainer"></div>

    <div class="interactive-alert-overlay" id="interactiveAlertModal">
        <div class="interactive-alert-box">
            <span class="material-icons interactive-alert-icon" id="interactiveAlertIcon">notifications_active</span>
            <div class="interactive-alert-title" id="interactiveAlertTitle">Aviso SegurApp</div>
            <div class="interactive-alert-msg" id="interactiveAlertMsg">Mensaje de prueba interactivo.</div>
            <div id="interactiveAlertInputContainer" style="display: none;">
                <input type="text" id="interactiveAlertInputField" class="interactive-alert-input" placeholder="Escribe aquí...">
            </div>
            <div class="interactive-alert-buttons" id="interactiveAlertButtons">
                <button class="interactive-alert-btn primary" id="interactiveAlertBtnOk" onclick="cerrarAlertaInteractiva(true)">Aceptar</button>
                <button class="interactive-alert-btn secondary" id="interactiveAlertBtnCancel" onclick="cerrarAlertaInteractiva(false)" style="display: none;">Cancelar</button>
            </div>
        </div>
    </div>

    <div class="background-overlay"></div>
    <div class="content"></div>

    <div class="user-top-bar" id="userTopBar">
        <div class="user-info-pill">
            <img src="https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/699116233_989437980648237_9201268186456313724_n.jpg?stp=dst-jpg_tt6&cstp=mx944x1135&ctp=s944x1135&_nc_cat=104&ccb=1-7&_nc_sid=6ee11a&_nc_eui2=AeF20nckf-W9D3ZFO7cEU1xSlECitdgceRmUQKK12Bx5GWvzlgUUxyauRT7-iPD92RZ-aTX-rvfv8ZiXQ66KkWHV&_nc_ohc=gxdvSRKUXnkQ7kNvwH8WqQ0&_nc_oc=AdqGX60bjXKBWj1ceGkZ8s_Nl-gyLmsiZyaXsmt-6h8ix8i-kmGDyJS8wE1suU0kd0A&_nc_zt=23&_nc_ht=scontent.fnva2-1.fna&_nc_gid=28yi0NLzXlWAn26Tp1SREA&_nc_ss=7b2a8&oh=00_AQGF3Tw8B2dUGXcmo646pQubZHhHwh9nsilv8PnxbB4ULQ&oe=6A8D0CAA" id="barUserAvatar" class="user-avatar-mini" alt="Avatar">
            <span id="barUserName">Invitado</span>
        </div>
        <div class="top-bar-buttons">
            <button class="btn-edit-trigger" id="btnEditUser" style="display: none;" onclick="abrirEditModal()">
                <span class="material-icons" style="font-size: 15px;">edit</span> <span>Editar</span>
            </button>
            <button class="btn-auth-trigger" id="btnAuthTrigger" onclick="abrirAuthModal()">
                <span class="material-icons" style="font-size: 16px;">account_circle</span> <span id="lblBtnAuth">Ingresar</span>
            </button>
        </div>
    </div>  

    <div class="hint-girar" onclick="voltearCarnet()">
        <span class="material-icons icono-girar">autorenew</span>
        Toca el carnet para voltear
    </div>

    <div class="escena-carnet" id="escenaCarnet">
        <div class="carnet-inner">
            
            <div class="carnet-cara frontal" onclick="voltearCarnet()">
                <div class="perforacion-lanyard"></div>

                <div class="carnet-header">
                    <div class="sub-brand">RAPIDOS - confiables - seguros.</div>
                    <span class="badge-rol">GERENTE DE OPERACIONES</span>
                </div>

                <div class="carnet-body">
                    <div class="foto-marco">
                        <img src=" https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/699116233_989437980648237_9201268186456313724_n.jpg?stp=dst-jpg_tt6&cstp=mx944x1135&ctp=s944x1135&_nc_cat=104&ccb=1-7&_nc_sid=6ee11a&_nc_eui2=AeF20nckf-W9D3ZFO7cEU1xSlECitdgceRmUQKK12Bx5GWvzlgUUxyauRT7-iPD92RZ-aTX-rvfv8ZiXQ66KkWHV&_nc_ohc=gxdvSRKUXnkQ7kNvwH8WqQ0&_nc_oc=AdqGX60bjXKBWj1ceGkZ8s_Nl-gyLmsiZyaXsmt-6h8ix8i-kmGDyJS8wE1suU0kd0A&_nc_zt=23&_nc_ht=scontent.fnva2-1.fna&_nc_gid=2ZhL4AnLEcgrYggw4zQ7wA&_nc_ss=7b2a8&oh=00_AQGikKLstA15EhhrMzhJ_lwprXU-tlo_gftb3RHt08PChg&oe=6A8DB56A" onerror="this.src='https://ui-avatars.com/api/?name=Sergio+Tapiero&background=ffd700&color=000&size=200'" alt="Foto Conductor" class="foto-conductor">
                        <div class="badge-verificado-icono">
                            <span class="material-icons" style="font-size: 18px;">check_circle</span>
                        </div>
                    </div>

                    <div class="nombre-conductor">Sergio Alejandro Tapiero Chala</div>
                    <span class="dato-val destacado">1.006.506.890</span>

                    <div class="cargo-conductor">SegurApp Recorridos</div>

                    <div class="datos-grid">
                        <div class="dato-item full-width">
                            <span class="dato-label">Rol Asignado</span>
                            <span class="dato-val destacado">Gerente de Operaciones</span>
                        </div>
                        <div class="dato-item">
                            <span class="dato-label">vehiculo registrado</span>
                            <span class="dato-val destacado">TVS Raider</span>
                        </div>
                        <div class="dato-item">
                            <span class="dato-label">Placa Vehículo</span>
                            <span class="dato-val destacado">BWQ 69H</span>
                        </div>
                    </div>

                    <div class="qr-destacado-container">
                        <div class="qr-code-lg">
                            <img src="https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/776224167_122117127363318735_1569553498169488314_n.jpg?stp=dst-jpg_tt6&cstp=mx2048x2048&ctp=s2048x2048&_nc_cat=106&ccb=1-7&_nc_sid=127cfc&_nc_ohc=acZEWJ2SiIMQ7kNvwEhGASd&_nc_oc=AdpsIKjJ4q091H4UtkOSX7o3HXi-fkS4nk_PFZKyzfoEabZrwtbMIMYUFwIOmDZ4hUk&_nc_zt=23&_nc_ht=scontent.fnva2-1.fna&_nc_gid=IaZSX4CnQsy-RUZYL4MaNg&_nc_ss=7b2a8&oh=00_AQFCn0EW_57uuAUNxx49t1Pw1eCsYqhurSWvv1KN2_k96Q&oe=6A8DD144" alt="Código QR WhatsApp">
                        </div>
                    </div>
                </div>

                <div class="carnet-footer">
                    <div class="info-qr">
                        <div class="id-codigo">ID: SEG-2026-890</div>
                        <div class="estado">
                            <span class="material-icons" style="font-size: 12px;">circle</span> Activo / Verificado
                        </div>
                    </div>
                </div> 
            </div>

            <div class="carnet-cara reverso" onclick="voltearCarnet()">
                <div class="perforacion-lanyard"></div>
                
                <div class="carnet-header">
                    <img src="https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/779323005_122117427681318735_2031315816040041680_n.jpg?stp=dst-jpg_tt6&cstp=mx2048x2048&ctp=s2048x2048&_nc_cat=111&ccb=1-7&_nc_sid=127cfc&_nc_ohc=wjqaYQyhMqMQ7kNvwHhbLRr&_nc_oc=AdqrwQYFrrpqb9myiexGiJLu0UjjBnxUyB-exAun21aULMQb_SIoZG81V4UJ-sVN-D8&_nc_zt=23&_nc_ht=scontent.fnva2-1.fna&_nc_gid=wOMHHuwyxKz37jCMtbLEEA&_nc_ss=7b2a8&oh=00_AQHmETtRU7Lh27W45hofDd7WgToFJYNvZzWw0d1mjd4M6A&oe=6A8DBBD2" alt="SegurApp Logo" class="logo" style="max-width: 250px;">
                    <div class="sub-brand">Información y Valoración</div>
                </div>

                <div class="reverso-info">
                    <p>Acredita el rol de <strong>Conductor Autorizado</strong> en <strong>SegurApp Recorridos</strong>.</p>
                    
                    <div class="rating-box-reverso" onclick="event.stopPropagation()">
                        <div class="rating-title">Calificación del Conductor</div>
                        <div class="rating-promedio">
                            <span id="promedioTexto">4.8</span> 
                            <span class="material-icons" style="font-size:19px; color:#d4af37;">star</span> 
                            <span id="totalVotosTexto" style="color:#666; font-size:0.78rem;">(80 valoraciones)</span>
                        </div>

                        <div class="stars-container" id="starsContainer">
                            <span class="material-icons star-icon" data-value="1" onclick="calificar(1)">star</span>
                            <span class="material-icons star-icon" data-value="2" onclick="calificar(2)">star</span>
                            <span class="material-icons star-icon" data-value="3" onclick="calificar(3)">star</span>
                            <span class="material-icons star-icon" data-value="4" onclick="calificar(4)">star</span>
                            <span class="material-icons star-icon" data-value="5" onclick="calificar(5)">star</span>
                        </div>

                        <div id="votoMensaje" class="voto-mensaje"></div>
                    </div>

                    <div class="datos-grid" style="margin-bottom: 8px; text-align: center;">
                        <div class="dato-item full-width">
                            <span class="dato-val destacado">LINEA DIRECTA DE ATENCIÓN</span>
                            <span class="dato-val">+57 318 988 2787</span>
                        </div>
                    </div>

                    <p>Este carnet acredita la identidad y el <strong>Rol de Conductor Autorizado</strong> en la plataforma <strong>SegurApp Recorridos</strong> en Neiva, Huila.</p>
                </div>

                <div style="margin-top: 6px; text-align: center; font-size: 0.75rem; color: #666;">
                    © 2026 SegurApp Recorridos. Todos los derechos reservados.
                </div>
            </div>

        </div>
    </div>

    <div class="acciones-bar">
        <button class="btn-accion solicitar-wasap" id="btnSolicitarRecorrido" onclick="enviarUbicacionPorWhatsApp()">
            <span class="material-icons" style="font-size:26px;">location_on</span> Solicitar Recorrido Ya  
        </button>

        <div class="acciones-bar">
            <button class="btn-accion" onclick="abrirDomicilioModal()" style="background: linear-gradient(135deg, var(--dorado-brillante) 0%, var(--dorado-oscuro) 100%); color: #000; font-weight: 900; text-transform: uppercase; border: none; box-shadow: 0 0 15px var(--glow-dorado);">
                <span class="material-icons" style="font-size: 22px;">local_shipping</span> Solicitar Servicio de Domicilios  
            </button>
            <a href="tel:3189882787" class="btn-accion" style="background: rgba(22, 25, 32, 0.9); color: #fff; font-weight: 900; text-transform: uppercase; border: 1px solid var(--borde-sutil);">
                <span class="material-icons" style="font-size: 22px;">phone</span> Llamar a Línea Directa  
            </a>
            <button class="btn-accion quienes-somos" onclick="abrirQuienesSomos()">
                <span class="material-icons">info</span> Nosotros
            </button>
            <button class="btn-accion terminos-condiciones" onclick="abrirTerminos()">
                <span class="material-icons">gavel</span> Términos
            </button>
        </div>
    </div>

    <div class="modal-overlay" id="modalAuth">
        <div class="modal-contenido">
            <div class="modal-header">
                <div class="modal-titulo">
                    <span class="material-icons">badge</span> Cuenta de Usuario
                </div>
                <button class="btn-cerrar" onclick="cerrarAuthModal()">
                    <span class="material-icons">close</span>
                </button>
            </div>

            <div class="auth-tabs">
                <button class="auth-tab-btn active" id="tabLogin" onclick="switchAuthTab('login')">Ingresar</button>
                <button class="auth-tab-btn" id="tabRegister" onclick="switchAuthTab('register')">Registro</button>
                <button class="auth-tab-btn" id="tabForgot" onclick="switchAuthTab('forgot')">Recuperar</button>
            </div>

            <form id="formLogin" onsubmit="procesarLogin(event)">
                <div class="form-group">
                    <label>Número de Cédula (CC)</label>
                    <input type="number" id="loginCC" required placeholder="Ej: 1006506890">
                </div>
                <div class="form-group">
                    <label>Clave</label>
                    <input type="password" id="loginPass" required placeholder="********">
                </div>
                <span class="link-forgot-pass" onclick="switchAuthTab('forgot')">¿Olvidaste tu contraseña?</span>
                <button type="submit" class="btn-submit-auth">Ingresar</button>
            </form>

            <form id="formRegister" style="display: none;" onsubmit="procesarRegistro(event)">
                <div class="avatar-upload-preview">
                    <img src="https://ui-avatars.com/api/?name=Foto&background=333&color=fff" id="previewFoto" class="preview-circle" alt="Preview">
                    <div style="flex: 1;">
                        <label style="font-size: 0.75rem; color: var(--dorado-brillante); font-weight: 700; text-transform: uppercase; display: block; margin-bottom: 4px;">Foto de Perfil</label>
                        <input type="file" id="regFoto" accept="image/*" onchange="convertirFotoBase64(this, 'previewFoto')" style="font-size: 0.75rem; color: #aaa; width: 100%;">
                    </div>
                </div>

                <div class="form-group">
                    <label>Nombre Completo *</label>
                    <input type="text" id="regNombre" required placeholder="Ej: Juan Pérez">
                </div>
                <div class="form-group">
                    <label>Cédula de Ciudadanía (CC) *</label>
                    <input type="number" id="regCC" required placeholder="Ej: 10064456470">
                </div>
                <div class="form-group">
                    <label>Número Celular *</label>
                    <input type="tel" id="regTel" required placeholder="Ej: 3189882787">
                </div>
                <div class="form-group">
                    <label>Contacto de Emergencia (Opcional)</label>
                    <input type="tel" id="regEmergencia" placeholder="Ej: 3150000000">
                </div>
                <div class="form-group">
                    <label>Clave *</label>
                    <input type="password" id="regPass" required placeholder="Crea una contraseña">
                </div>
                <button type="submit" class="btn-submit-auth">Crear Mi Cuenta</button>
            </form>

            <form id="formForgot" style="display: none;" onsubmit="procesarRecuperacion(event)">
                <p style="font-size:0.8rem; color:#ccc; margin-bottom:12px;">
                    Ingresa tu número de Cédula y el Teléfono Celular asociado a tu cuenta para restaurar tu contraseña.
                </p>
                <div class="form-group">
                    <label>Cédula de Ciudadanía (CC) *</label>
                    <input type="number" id="forgotCC" required placeholder="Ej: 1006506890">
                </div>
                <div class="form-group">
                    <label>Número Celular Registrado *</label>
                    <input type="tel" id="forgotTel" required placeholder="Ej: 3189882787">
                </div>
                <div class="form-group">
                    <label>Nueva Contraseña *</label>
                    <input type="password" id="forgotNewPass" required placeholder="Nueva clave">
                </div>
                <button type="submit" class="btn-submit-auth">Restablecer Clave</button>
            </form>
        </div>
    </div>

    <div class="modal-overlay" id="modalDomicilio">
        <div class="modal-contenido">
            <div class="modal-header">
                <div class="modal-titulo">
                    <span class="material-icons">local_shipping</span> Solicitar Domicilio
                </div>
                <button class="btn-cerrar" onclick="cerrarDomicilioModal()">
                    <span class="material-icons">close</span>
                </button>
            </div>

            <form id="formDomicilio" onsubmit="enviarDomicilioWhatsApp(event)">
                <div class="form-group">
                    <label>Lugar de Recogida o Negocio *</label>
                    <input type="text" id="domLugar" required placeholder="Ej: Restaurante El Fogón / Tienda Centro">
                </div>
                <div class="form-group">
                    <label>Detalles del Pedido / Producto a Reclamar *</label>
                    <textarea id="domDetalles" rows="3" required placeholder="Ej: 2 almuerzos ejecutivos, pago pendiente o factura #123"></textarea>
                </div>
                <div class="form-group">
                    <label>Dirección de Entrega Final Cliente *</label>
                    <input type="text" id="domDireccion" required placeholder="Ej: Calle 8 # 15-20, Apto 302">
                </div>
                <div class="form-group">
                    <label>Teléfono de la Persona que Recibe *</label>
                    <input type="tel" id="domTelefono" required placeholder="Ej: 3101234567">
                </div>
                <button type="submit" class="btn-submit-auth" style="background: linear-gradient(135deg, #00ff88 0%, #25d366 100%); color: #000;">
                    Enviar Domicilio por WhatsApp
                </button>
            </form>
        </div>
    </div>

    <div class="modal-overlay" id="modalEditUser">
        <div class="modal-contenido">
            <div class="modal-header">
                <div class="modal-titulo">
                    <span class="material-icons">manage_accounts</span> Editar Perfil
                </div>
                <button class="btn-cerrar" onclick="cerrarEditModal()">
                    <span class="material-icons">close</span>
                </button>
            </div>

            <form id="formEditUser" onsubmit="procesarEdicionUsuario(event)">
                <div class="avatar-upload-preview">
                    <img src="https://ui-avatars.com/api/?name=Foto&background=333&color=fff" id="previewFotoEdit" class="preview-circle" alt="Preview Edit">
                    <div style="flex: 1;">
                        <label style="font-size: 0.75rem; color: var(--dorado-brillante); font-weight: 700; text-transform: uppercase; display: block; margin-bottom: 4px;">Cambiar Foto</label>
                        <input type="file" id="editFoto" accept="image/*" onchange="convertirFotoBase64(this, 'previewFotoEdit')" style="font-size: 0.75rem; color: #aaa; width: 100%;">
                    </div>
                </div>

                <div class="form-group">
                    <label>Nombre Completo</label>
                    <input type="text" id="editNombre" required>
                </div>
                <div class="form-group">
                    <label>Cédula (No editable)</label>
                    <input type="number" id="editCC" readonly style="opacity: 0.6; cursor: not-allowed;">
                </div>
                <div class="form-group">
                    <label>Número Celular</label>
                    <input type="tel" id="editTel" required>
                </div>
                <div class="form-group">
                    <label>Contacto de Emergencia</label>
                    <input type="tel" id="editEmergencia">
                </div>
                <div class="form-group">
                    <label>Nueva Clave (Opcional)</label>
                    <input type="password" id="editPass" placeholder="Dejar en blanco si no deseas cambiarla">
                </div>
                <button type="submit" class="btn-submit-auth">Guardar Cambios</button>
            </form>
        </div>
    </div>

    <div class="modal-overlay" id="modalQuienesSomos">
        <div class="modal-contenido">
            <div class="modal-header">
                <div class="modal-titulo">
                    <span class="material-icons">groups</span> Quiénes Somos
                </div>
                <button class="btn-cerrar" onclick="cerrarQuienesSomos()">
                    <span class="material-icons">close</span>
                </button>
            </div>
            <div class="modal-body">
                <p>En <strong>SegurApp Recorridos</strong> nos dedicamos a transformar la movilización urbana en Neiva, ofreciendo un servicio de transporte individual express altamente seguro, rápido y confiable.</p>
                
                <h4><span class="material-icons">verified_user</span> Nuestra Misión</h4>
                <p>Proporcionar un traslado rápido y seguro para cada pasajero, respaldado por conductores totalmente identificados y verificados en tiempo real.</p>

                <h4><span class="material-icons">star</span> ¿Por qué elegirnos?</h4>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Seguridad Garantizada:</strong> Validación digital de la identidad del conductor e historial de vehículo.
                </div>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Respuesta Inmediata:</strong> Ubicación precisa en tiempo real a través de WhatsApp para recogidas inmediatas.
                </div>
            </div>
        </div>
    </div>

    <div class="modal-overlay" id="modalTerminos">
        <div class="modal-contenido">
            <div class="modal-header">
                <div class="modal-titulo" style="display: flex; align-items: center; gap: 10px;">
                    <img src="https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/762404541_122113498089318735_2648448917068400992_n.jpg?stp=dst-jpg_tt6&cstp=mx2048x2048&ctp=s2048x2048&_nc_cat=104&ccb=1-7&_nc_sid=127cfc&_nc_ohc=mGmYOOWmg8sQ7kNvwHgJb_U&_nc_oc=AdriBDFiD1RlWEULfa4DVGG6Dbrn5mG_UmGyDQQYAAzDg8gu7oCQ6Qupb9gV-bcomrk&_nc_zt=23&_nc_ht=scontent.fnva2-1.fna&_nc_gid=i8rHeoTssz7CXab1o0kxzQ&_nc_ss=7b2a8&oh=00_AQESo0KmB5qrtJ-xKX2OBC7X-Fr8iTdkf5IouiQT_9t5gA&oe=6A8DB87F" alt="SegurApp Logo" style="height: 50px; width: auto; object-fit: contain;">
                    <span>Términos y Condiciones</span>
                </div>
                <button class="btn-cerrar" onclick="cerrarTerminos()">
                    <span class="material-icons">close</span>
                </button>
            </div>
            <div class="modal-body">
                <div class="alerta-seguridad-box">
                    <div class="alerta-header">
                        <div class="alerta-titulo-text">
                            <span class="material-icons" style="font-size: 20px;">shield</span> Protocolo Neiva
                        </div>
                        <div class="badge-live-seguridad">
                            <span class="dot-live"></span> Activo
                        </div>
                    </div>
                    <p class="alerta-body-desc">
                        Diseñado prioritariamente para proteger la integridad física y patrimonial del usuario y del conductor ante eventualidades de seguridad urbana.
                    </p>
                    <div class="alerta-puntos-claves">
                        <div class="punto-seguridad-item">
                            <span class="material-icons">my_location</span> Geolocalización activa requerida
                        </div>
                        <div class="punto-seguridad-item">
                            <span class="material-icons">badge</span> Verificación mutua de identidad
                        </div>
                        <div class="punto-seguridad-item">
                            <span class="material-icons">health_and_safety</span> Casco de protección obligatorio
                        </div>
                    </div>
                </div>

                <h4 class="terminos-titulo-seccion"><span class="material-icons">verified</span> 1. Identificación y Verificación Mutual</h4>
                <p><strong>• Del Conductor:</strong> Se garantiza la identidad del conductor (Sergio Alejandro Tapiero Chala - C.C. 1.006.506.890) y del vehículo registrado (TVS Raider Negra - BWQ 69H).</p>
                <p><strong>• Del Pasajero:</strong> El usuario debe proporcionar voluntariamente su ubicación GPS exacta en tiempo real antes de abordar. El conductor se reserva el derecho de verificar la identidad del pasajero (Cédula de Ciudadanía) antes de iniciar la marcha.</p>

                <h4 class="terminos-titulo-seccion"><span class="material-icons">security</span> 2. Protocolo de Seguridad del Conductor</h4>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Cancelación por Inseguridad:</strong> El conductor puede rechazar o cancelar cualquier servicio si el punto de recogida o destino representa un riesgo inminente, zonas de alteración de orden público o barrios con restricciones operativas de seguridad.
                </div>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Inspección Prevención:</strong> No se transportarán elementos sospechosos, maletas cerradas de procedencia dudosa, ni paquetes de gran volumen que comprometan la maniobrabilidad de la motocicleta o la seguridad vial.
                </div>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Estado del Usuario:</strong> Se prohíbe rotundamente el traslado de personas bajo sospecha de porte de armas, sustancias psicoactivas o en estado de embri embriaguez extrema que pongan en riesgo la estabilidad del vehículo.
                </div>

                <h4 class="terminos-titulo-seccion"><span class="material-icons">health_and_safety</span> 3. Protocolo de Seguridad del Cliente (Pasajero)</h4>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Casco Obligatorio:</strong> Uso indispensable y correcto del casco de protección suministrado o propio durante todo el recorrido.
                </div>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Rastreabilidad:</strong> Cada servicio genera un enlace directo de WhatsApp con coordenadas GPS que el pasajero puede compartir con familiares en tiempo real.
                </div>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Monitoreo de Ruta:</strong> El recorrido seguirá estrictamente la ruta navegable más segura. No se realizarán desvíos no autorizados solicitados en la vía sin previa verificación de seguridad.
                </div>

                <h4 class="terminos-titulo-seccion"><span class="material-icons">dns</span> 4. Registro de Datos y Manejo de Información</h4>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Recolección y Autorización:</strong> Al registrarse, crear una cuenta o utilizar los servicios de <strong>SegurApp Recorridos</strong>, el usuario autoriza de manera expresa la recolección, almacenamiento y tratamiento de sus datos personales.
                </div>
                <div class="caracteristica-box">
                    <strong style="color: #fff;">• Finalidad del Tratamiento:</strong> La información recopilada tiene como única finalidad garantizar la seguridad operativa de los trayectos, validar la identidad mutua y facilitar la comunicación inmediata.
                </div>

                <button class="btn-aceptar-terminos" onclick="aceptarTerminosModal()">
                    <span class="material-icons">check_circle</span> Aceptar y Continuar
                </button>
            </div>
        </div>
    </div>

    <script>
        const STORAGE_KEY_VOTO = 'segurapp_user_voted';
        const STORAGE_KEY_START = 'segurapp_start_time';
        const STORAGE_KEY_TERMS = 'segurapp_terms_accepted_v1';
        const STORAGE_USERS = 'segurapp_registered_users';
        const STORAGE_CURRENT_USER = 'segurapp_current_logged_user';
        const STORAGE_RATINGS_DATA = 'segurapp_ratings_data_v1';
        
        const URL_APPS_SCRIPT = "https://script.google.com/macros/s/AKfycbxFfdvKCpwciO7odqxyqredyQWGf1MEvIh7CIZts8PoEySLxMxIxbqG51ST5_qEnvaF/exec";

        let fotoBase64Temp = '';

        // --- SISTEMA DE AUDIO PARA BOTONES ---
        function reproducirSonidoBoton() {
            try {
                const audioCtx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = audioCtx.createOscillator();
                const gain = audioCtx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, audioCtx.currentTime); // Nota D5 agradable
                osc.frequency.exponentialRampToValueAtTime(880, audioCtx.currentTime + 0.05);
                gain.gain.setValueAtTime(0.08, audioCtx.currentTime);
                gain.gain.exponentialRampToValueAtTime(0.001, audioCtx.currentTime + 0.05);
                osc.connect(gain);
                gain.connect(audioCtx.destination);
                osc.start();
                osc.stop(audioCtx.currentTime + 0.05);
            } catch (e) {
                // Previene errores de políticas de audio del navegador antes de la primera interacción
            }
        }

        // Se asigna sonido automáticamente a todos los botones, enlaces de acción, pestañas y estrellas
        document.addEventListener('DOMContentLoaded', () => {
            const selectorBotones = 'button, .btn-accion, .btn-edit-trigger, .btn-auth-trigger, .auth-tab-btn, .btn-submit-auth, .btn-cerrar, .interactive-alert-btn, .btn-aceptar-terminos, .star-icon, .hint-girar';
            document.addEventListener('click', (e) => {
                const elemento = e.target.closest(selectorBotones);
                if (elemento) {
                    reproducirSonidoBoton();
                }
            });
        });

        function mostrarToast(titulo, mensaje, tipo = 'success') {
            const container = document.getElementById('toastContainer');
            const toast = document.createElement('div');
            toast.className = `custom-toast ${tipo}`;
            
            let icono = 'check_circle';
            if (tipo === 'error') icono = 'error';
            else if (tipo === 'info') icono = 'info';

            toast.innerHTML = `
                <span class="material-icons toast-icon">${icono}</span>
                <div class="toast-content">
                    <span class="toast-title">${titulo}</span>
                    <span class="toast-msg">${mensaje}</span>
                </div>
                <button class="toast-close" onclick="this.parentElement.remove()">&times;</button>
            `;

            container.appendChild(toast);
            setTimeout(() => { toast.classList.add('show'); }, 50);

            setTimeout(() => {
                toast.classList.remove('show');
                setTimeout(() => { toast.remove(); }, 400);
            }, 4000);
        }

        let resolveAlertaInteractiva = null;

        function mostrarAlertaInteractiva({ titulo, mensaje, tipo = 'info', esPrompt = false, esConfirm = false, valorInicial = '' }) {
            return new Promise((resolve) => {
                resolveAlertaInteractiva = resolve;
                const overlay = document.getElementById('interactiveAlertModal');
                const titleEl = document.getElementById('interactiveAlertTitle');
                const msgEl = document.getElementById('interactiveAlertMsg');
                const iconEl = document.getElementById('interactiveAlertIcon');
                const inputContainer = document.getElementById('interactiveAlertInputContainer');
                const inputField = document.getElementById('interactiveAlertInputField');
                const btnOk = document.getElementById('interactiveAlertBtnOk');
                const btnCancel = document.getElementById('interactiveAlertBtnCancel');

                titleEl.innerText = titulo;
                msgEl.innerText = mensaje;

                if (tipo === 'success') iconEl.innerText = 'check_circle';
                else if (tipo === 'error') iconEl.innerText = 'error';
                else iconEl.innerText = 'notifications_active';

                if (esPrompt) {
                    inputContainer.style.display = 'block';
                    inputField.value = valorInicial;
                    btnCancel.style.display = 'block';
                    btnOk.innerText = 'Aceptar';
                    setTimeout(() => inputField.focus(), 100);
                } else if (esConfirm) {
                    inputContainer.style.display = 'none';
                    btnCancel.style.display = 'block';
                    btnOk.innerText = 'Sí, Continuar';
                } else {
                    inputContainer.style.display = 'none';
                    btnCancel.style.display = 'none';
                    btnOk.innerText = 'Aceptar';
                }

                overlay.classList.add('active');
            });
        }

        function cerrarAlertaInteractiva(resultado) {
            const overlay = document.getElementById('interactiveAlertModal');
            const inputField = document.getElementById('interactiveAlertInputField');
            overlay.classList.remove('active');

            if (resolveAlertaInteractiva) {
                const inputContainer = document.getElementById('interactiveAlertInputContainer');
                if (inputContainer.style.display === 'block') {
                    resolveAlertaInteractiva(resultado ? inputField.value : null);
                } else {
                    resolveAlertaInteractiva(resultado);
                }
                resolveAlertaInteractiva = null;
            }
        }

        function obtenerUsuarios() {
            return JSON.parse(localStorage.getItem(STORAGE_USERS) || '[]');
        }

        function obtenerUsuarioActual() {
            return JSON.parse(localStorage.getItem(STORAGE_CURRENT_USER) || 'null');
        }

        function switchAuthTab(type) {
            const btnLogin = document.getElementById('tabLogin');
            const btnRegister = document.getElementById('tabRegister');
            const btnForgot = document.getElementById('tabForgot');
            
            const formLogin = document.getElementById('formLogin');
            const formRegister = document.getElementById('formRegister');
            const formForgot = document.getElementById('formForgot');

            btnLogin.classList.remove('active');
            btnRegister.classList.remove('active');
            btnForgot.classList.remove('active');

            formLogin.style.display = 'none';
            formRegister.style.display = 'none';
            formForgot.style.display = 'none';

            if (type === 'login') {
                btnLogin.classList.add('active');
                formLogin.style.display = 'block';
            } else if (type === 'register') {
                btnRegister.classList.add('active');
                formRegister.style.display = 'block';
            } else if (type === 'forgot') {
                btnForgot.classList.add('active');
                formForgot.style.display = 'block';
            }
        }

        function convertirFotoBase64(input, targetImgId) {
            if (input.files && input.files[0]) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    fotoBase64Temp = e.target.result;
                    document.getElementById(targetImgId).src = fotoBase64Temp;
                };
                reader.readAsDataURL(input.files[0]);
            }
        }

        async function guardarEnGoogleSheets(datosUsuario, accion) {
            try {
                await fetch(URL_APPS_SCRIPT, {
                    method: 'POST',
                    mode: 'no-cors',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ accion: accion, ...datosUsuario })
                });
            } catch (error) {
                console.error("Error al sincronizar con Google Sheets:", error);
            }
        }

        async function procesarRegistro(e) {
            e.preventDefault();
            const nombre = document.getElementById('regNombre').value.trim();
            const cc = document.getElementById('regCC').value.trim();
            const tel = document.getElementById('regTel').value.trim();
            const emergencia = document.getElementById('regEmergencia').value.trim();
            const pass = document.getElementById('regPass').value.trim();

            const usuarios = obtenerUsuarios();
            if (usuarios.some(u => u.cc === cc)) {
                mostrarToast('Error de Registro', 'Ya existe un usuario registrado con esta cédula (CC).', 'error');
                return;
            }

            const fotoFinal = fotoBase64Temp || `https://ui-avatars.com/api/?name=${encodeURIComponent(nombre)}&background=ffd700&color=000`;
            const nuevoUsuario = { nombre, cc, tel, emergencia, pass, foto: fotoFinal };

            usuarios.push(nuevoUsuario);
            localStorage.setItem(STORAGE_USERS, JSON.stringify(usuarios));
            localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(nuevoUsuario));

            await guardarEnGoogleSheets(nuevoUsuario, 'registro');

            fotoBase64Temp = '';
            mostrarToast('¡Éxito!', '¡Cuenta creada y guardada exitosamente!', 'success');
            cerrarAuthModal();
            cargarEstadoUsuario();
        }

        function procesarLogin(e) {
            e.preventDefault();
            const cc = document.getElementById('loginCC').value.trim();
            const pass = document.getElementById('loginPass').value.trim();

            if (cc === "1006506890" && (pass === "0408" || pass === "admin" || pass === "Sergio2026")) {
                const driverUser = {
                    nombre: "Sergio Alejandro Tapiero Chala",
                    cc: "1006506890",
                    tel: "3189882787",
                    emergencia: "3150000000",
                    pass: pass,
                    foto: "https://scontent.fnva2-1.fna.fbcdn.net/v/t39.30808-6/699116233_989437980648237_9201268186456313724_n.jpg"
                };
                localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(driverUser));
                mostrarToast('Bienvenido', '¡Bienvenido, Conductor Oficial Sergio Tapiero!', 'success');
                cerrarAuthModal();
                cargarEstadoUsuario();
                return;
            }

            const usuarios = obtenerUsuarios();
            const user = usuarios.find(u => u.cc === cc && u.pass === pass);

            if (user) {
                localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(user));
                mostrarToast('Bienvenido', `¡Bienvenido de nuevo, ${user.nombre}!`, 'success');
                cerrarAuthModal();
                cargarEstadoUsuario();
            } else {
                mostrarToast('Acceso Denegado', 'Cédula o contraseña incorrectas.', 'error');
            }
        }

        async function procesarRecuperacion(e) {
            e.preventDefault();
            const cc = document.getElementById('forgotCC').value.trim();
            const tel = document.getElementById('forgotTel').value.trim();
            const newPass = document.getElementById('forgotNewPass').value.trim();

            let usuarios = obtenerUsuarios();
            const index = usuarios.findIndex(u => u.cc === cc && u.tel === tel);

            if (index !== -1) {
                usuarios[index].pass = newPass;
                localStorage.setItem(STORAGE_USERS, JSON.stringify(usuarios));
                
                const currentUser = obtenerUsuarioActual();
                if (currentUser && currentUser.cc === cc) {
                    currentUser.pass = newPass;
                    localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(currentUser));
                }

                await guardarEnGoogleSheets(usuarios[index], 'actualizar');

                mostrarToast('Restauración Exitosa', '¡Tu contraseña ha sido restablecida exitosamente!', 'success');
                switchAuthTab('login');
                document.getElementById('formForgot').reset();
            } else {
                mostrarToast('Error', 'No se encontró ningún usuario que coincida con esa cédula y teléfono.', 'error');
            }
        }

        function abrirEditModal() {
            const user = obtenerUsuarioActual();
            if (!user) return;

            document.getElementById('editNombre').value = user.nombre;
            document.getElementById('editCC').value = user.cc;
            document.getElementById('editTel').value = user.tel;
            document.getElementById('editEmergencia').value = user.emergencia || '';
            document.getElementById('previewFotoEdit').src = user.foto;
            document.getElementById('editPass').value = '';
            fotoBase64Temp = '';

            document.getElementById('modalEditUser').classList.add('active');
        }

        function cerrarEditModal() {
            document.getElementById('modalEditUser').classList.remove('active');
        }

        async function procesarEdicionUsuario(e) {
            e.preventDefault();
            const currentUser = obtenerUsuarioActual();
            if (!currentUser) return;

            const nombre = document.getElementById('editNombre').value.trim();
            const tel = document.getElementById('editTel').value.trim();
            const emergencia = document.getElementById('editEmergencia').value.trim();
            const pass = document.getElementById('editPass').value.trim();

            let usuarios = obtenerUsuarios();
            const index = usuarios.findIndex(u => u.cc === currentUser.cc);

            if (index !== -1) {
                usuarios[index].nombre = nombre;
                usuarios[index].tel = tel;
                usuarios[index].emergencia = emergencia;
                if (fotoBase64Temp) usuarios[index].foto = fotoBase64Temp;
                if (pass) usuarios[index].pass = pass;

                localStorage.setItem(STORAGE_USERS, JSON.stringify(usuarios));
                localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(usuarios[index]));
                await guardarEnGoogleSheets(usuarios[index], 'actualizar');
            } else {
                currentUser.nombre = nombre;
                currentUser.tel = tel;
                currentUser.emergencia = emergencia;
                if (fotoBase64Temp) currentUser.foto = fotoBase64Temp;
                if (pass) currentUser.pass = pass;
                localStorage.setItem(STORAGE_CURRENT_USER, JSON.stringify(currentUser));
            }

            fotoBase64Temp = '';
            mostrarToast('Perfil Actualizado', '¡Perfil actualizado con éxito!', 'success');
            cerrarEditModal();
            cargarEstadoUsuario();
        }

        function cargarEstadoUsuario() {
            const user = obtenerUsuarioActual();
            const barUserName = document.getElementById('barUserName');
            const barUserAvatar = document.getElementById('barUserAvatar');
            const lblBtnAuth = document.getElementById('lblBtnAuth');
            const btnEditUser = document.getElementById('btnEditUser');

            if (user) {
                barUserName.innerText = user.nombre.split(' ')[0];
                barUserAvatar.src = user.foto;
                lblBtnAuth.innerText = 'Salir';
                btnEditUser.style.display = 'flex';
            } else {
                barUserName.innerText = 'Invitado';
                barUserAvatar.src = 'https://ui-avatars.com/api/?name=Usuario&background=333&color=fff';
                lblBtnAuth.innerText = 'Ingresar';
                btnEditUser.style.display = 'none';
            }
            
            inicializarSistemaCalificacion();
        }

        async function abrirAuthModal() {
            const user = obtenerUsuarioActual();
            if (user) {
                const confirmarCierre = await mostrarAlertaInteractiva({
                    titulo: 'Cerrar Sesión',
                    mensaje: `¿Deseas cerrar la sesión de ${user.nombre}?`,
                    tipo: 'info',
                    esConfirm: true
                });
                if (confirmarCierre) {
                    localStorage.removeItem(STORAGE_CURRENT_USER);
                    cargarEstadoUsuario();
                    mostrarToast('Sesión Cerrada', 'Has cerrado sesión correctamente.', 'success');
                }
            } else {
                switchAuthTab('login');
                document.getElementById('modalAuth').classList.add('active');
            }
        }

        function cerrarAuthModal() {
            document.getElementById('modalAuth').classList.remove('active');
        }

        function abrirDomicilioModal() {
            document.getElementById('modalDomicilio').classList.add('active');
        }

        function cerrarDomicilioModal() {
            document.getElementById('modalDomicilio').classList.remove('active');
        }

        function enviarDomicilioWhatsApp(e) {
            e.preventDefault();
            const lugar = document.getElementById('domLugar').value.trim();
            const detalles = document.getElementById('domDetalles').value.trim();
            const direccion = document.getElementById('domDireccion').value.trim();
            const telefono = document.getElementById('domTelefono').value.trim();

            const telefonoConductor = "+573189882787";
            const usuarioActual = obtenerUsuarioActual();
            const solicitante = usuarioActual ? usuarioActual.nombre : "Cliente General";

            let mensaje = `📦 *SOLICITUD DE SERVICIO DE DOMICILIO* - *SegurApp*\n\n`;
            mensaje += `👤 *Solicita:* ${solicitante}\n`;
            mensaje += `🏪 *Lugar de recogida / Negocio:* ${lugar}\n`;
            mensaje += `📝 *Detalles del pedido / Producto:* ${detalles}\n`;
            mensaje += `📍 *Dirección de entrega final:* ${direccion}\n`;
            mensaje += `📞 *Teléfono de quien recibe:* ${telefono}\n\n`;
            mensaje += `_Por favor confirmar la asignación del domicilio._`;

            const urlWhatsApp = `https://api.whatsapp.com/send?phone=${telefonoConductor}&text=${encodeURIComponent(mensaje)}`;
            window.open(urlWhatsApp, '_blank');
            cerrarDomicilioModal();
            mostrarToast('Domicilio Generado', 'Redirigiendo a WhatsApp con los detalles de tu domicilio.', 'success');
        }

        function obtenerDatosCalificacion() {
            const dataDefault = { sumaVotos: 384, totalVotos: 80, votosUsuarios: {} };
            const guardado = localStorage.getItem(STORAGE_RATINGS_DATA);
            if (guardado) {
                try { return JSON.parse(guardado); } catch(e) { return dataDefault; }
            }
            return dataDefault;
        }

        function guardarDatosCalificacion(data) {
            localStorage.setItem(STORAGE_RATINGS_DATA, JSON.stringify(data));
        }

        function inicializarSistemaCalificacion() {
            const data = obtenerDatosCalificacion();
            const promedio = data.totalVotos > 0 ? (data.sumaVotos / data.totalVotos).toFixed(1) : "4.8";
            
            document.getElementById('promedioTexto').innerText = promedio;
            document.getElementById('totalVotosTexto').innerText = `(${data.totalVotos} valoraciones)`;

            const usuarioActual = obtenerUsuarioActual();
            const stars = document.querySelectorAll('#starsContainer .star-icon');
            const mensajeEl = document.getElementById('votoMensaje');

            stars.forEach(s => s.classList.remove('active'));

            if (usuarioActual && data.votosUsuarios && data.votosUsuarios[usuarioActual.cc]) {
                const miVoto = data.votosUsuarios[usuarioActual.cc];
                destacarEstrellas(miVoto);
                mensajeEl.innerText = `¡Ya calificaste con ${miVoto} estrellas! ⭐`;
            } else if (localStorage.getItem(STORAGE_KEY_VOTO)) {
                const miVotoLocal = parseInt(localStorage.getItem(STORAGE_KEY_VOTO));
                destacarEstrellas(miVotoLocal);
                mensajeEl.innerText = `¡Calificación registrada: ${miVotoLocal} estrellas! ⭐`;
            } else {
                mensajeEl.innerText = '¡Toca una estrella para calificar!';
            }
        }

        function destacarEstrellas(cantidad) {
            const stars = document.querySelectorAll('#starsContainer .star-icon');
            stars.forEach((star, index) => {
                if (index < cantidad) {
                    star.classList.add('active');
                } else {
                    star.classList.remove('active');
                }
            });
        }

        async function calificar(estrellas) {
            const usuarioActual = obtenerUsuarioActual();

            if (!usuarioActual) {
                const irALogin = await mostrarAlertaInteractiva({
                    titulo: 'Aviso: Iniciar Sesión Requerido',
                    mensaje: 'Para garantizar valoraciones reales y seguras en SegurApp, debes iniciar sesión o registrarte antes de calificar al conductor.',
                    tipo: 'info',
                    esConfirm: true
                });
                if (irALogin) {
                    abrirAuthModal();
                }
                return;
            }

            let data = obtenerDatosCalificacion();
            if (!data.votosUsuarios) data.votosUsuarios = {};

            const votoAnterior = data.votosUsuarios[usuarioActual.cc];

            if (votoAnterior) {
                data.sumaVotos = data.sumaVotos - votoAnterior + estrellas;
                data.votosUsuarios[usuarioActual.cc] = estrellas;
                guardarDatosCalificacion(data);
                
                destacarEstrellas(estrellas);
                const promedio = (data.sumaVotos / data.totalVotos).toFixed(1);
                document.getElementById('promedioTexto').innerText = promedio;
                document.getElementById('votoMensaje').innerText = `¡Has actualizado tu calificación a ${estrellas} estrellas! ⭐`;
                
                mostrarToast('Calificación Actualizada', `Tu nueva valoración de ${estrellas} estrellas ha sido guardada.`, 'success');
            } else {
                data.sumaVotos += estrellas;
                data.totalVotos += 1;
                data.votosUsuarios[usuarioActual.cc] = estrellas;
                guardarDatosCalificacion(data);

                localStorage.setItem(STORAGE_KEY_VOTO, estrellas);

                destacarEstrellas(estrellas);
                const promedio = (data.sumaVotos / data.totalVotos).toFixed(1);
                document.getElementById('promedioTexto').innerText = promedio;
                document.getElementById('totalVotosTexto').innerText = `(${data.totalVotos} valoraciones)`;
                document.getElementById('votoMensaje').innerText = `¡Gracias por calificar con ${estrellas} estrellas! ⭐`;

                await mostrarAlertaInteractiva({
                    titulo: '¡Calificación Exitosa!',
                    mensaje: `Agradecemos tu valoración de ${estrellas} estrellas. Tu opinión ayuda a mantener la excelencia y seguridad en las rutas de SegurApp en Neiva.`,
                    tipo: 'success'
                });

                mostrarToast('¡Gracias!', 'Tu valoración ha sido registrada con éxito.', 'success');
            }
        }

        function voltearCarnet() {
            const escena = document.getElementById('escenaCarnet');
            escena.classList.toggle('flipped');
        }

        function enviarMensajeWhatsAppBase(telefonoConductor, nombreUsuario, telefonoUsuario, cedulaUsuario, enlaceMapa, sinUbicacion = false) {
            let mensaje = `¡Hola Sergio! 👋 Solicito un recorrido seguro con *SegurApp Recorridos*.\n\n`;
            mensaje += `👤 *Pasajero:* ${nombreUsuario}\n`;
            mensaje += `📱 *Celular:* ${telefonoUsuario}\n`;
            mensaje += `🆔 *Cédula:* ${cedulaUsuario}\n`;
            
            if (sinUbicacion) {
                mensaje += `📍 *Ubicación:* No se pudo acceder al GPS automáticamente, *Porfavor enviar Ubicacion en tiempo real, para confirmar la solicitud del servicio.*\n`;
            } else {
                mensaje += `📍 *Mi ubicación GPS en tiempo real:* ${enlaceMapa}\n`;
            }

            mensaje += `\n_Por favor confirmar disponibilidad de servicio._`;

            const urlWhatsApp = `https://api.whatsapp.com/send?phone=${telefonoConductor}&text=${encodeURIComponent(mensaje)}`;
            window.open(urlWhatsApp, '_blank');
        }

        function enviarUbicacionPorWhatsApp() {
            const telefonoConductor = "+573189882787";
            const usuarioActual = obtenerUsuarioActual();

            const nombreUsuario = usuarioActual ? usuarioActual.nombre : "Invitado / Pasajero";
            const telefonoUsuario = usuarioActual ? usuarioActual.tel : "No registrado";
            const cedulaUsuario = usuarioActual ? usuarioActual.cc : "N/A";

            if (!navigator.geolocation) {
                enviarMensajeWhatsAppBase(telefonoConductor, nombreUsuario, telefonoUsuario, cedulaUsuario, null, true);
                return;
            }

            mostrarToast("Geolocalización", "Obteniendo tu ubicación en tiempo real...", "info");

            navigator.geolocation.getCurrentPosition(
                (position) => {
                    const lat = position.coords.latitude;
                    const lng = position.coords.longitude;
                    const enlaceMapa = `https://www.google.com/maps?q=${lat},${lng}`;
                    enviarMensajeWhatsAppBase(telefonoConductor, nombreUsuario, telefonoUsuario, cedulaUsuario, enlaceMapa, false);
                },
                (error) => {
                    console.warn("Error obteniendo ubicación:", error);
                    mostrarToast("Aviso GPS", "No se pudo obtener la ubicación exacta. Enviando mensaje sin GPS.", "info");
                    enviarMensajeWhatsAppBase(telefonoConductor, nombreUsuario, telefonoUsuario, cedulaUsuario, null, true);
                },
                { enableHighAccuracy: true, timeout: 10000, maximumAge: 0 }
            );
        }

        function abrirQuienesSomos() {
            document.getElementById('modalQuienesSomos').classList.add('active');
        }

        function cerrarQuienesSomos() {
            document.getElementById('modalQuienesSomos').classList.remove('active');
        }

        function abrirTerminos() {
            document.getElementById('modalTerminos').classList.add('active');
        }

        function cerrarTerminos() {
            document.getElementById('modalTerminos').classList.remove('active');
        }

        function aceptarTerminosModal() {
            localStorage.setItem(STORAGE_KEY_TERMS, 'true');
            cerrarTerminos();
            mostrarToast("Términos", "¡Términos aceptados correctamente!", "success");
        }

        window.onload = function() {
            cargarEstadoUsuario();
        };
    </script>
</body>
</html>
