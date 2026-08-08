<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
    <title>Carlitos Móvil</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
    <script src="https://unpkg.com/html5-qrcode"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-database-compat.js"></script>
    <style>
        body { font-family: system-ui, -apple-system, sans-serif; background: #f3f4f6; margin: 0; padding-bottom: 80px; touch-action: manipulation; }
        .header { background: #dc2626; color: white; padding: 16px; text-align: center; font-weight: bold; font-size: 22px; box-shadow: 0 4px 6px rgba(0,0,0,0.1); position: sticky; top: 0; z-index: 50; }
        .btn-cart { background: #1f2937; color: white; padding: 14px; border-radius: 16px; font-weight: bold; display: flex; align-items: center; gap: 10px; justify-content: center; margin: 10px 15px; }
        .product-card { background: white; border-radius: 12px; padding: 12px 16px; box-shadow: 0 2px 4px rgba(0,0,0,0.05); margin-bottom: 10px; display: flex; justify-content: space-between; align-items: center; border: 1px solid #e5e7eb; }
        .cart-item { background: white; border-radius: 12px; padding: 10px 15px; margin-bottom: 8px; display: flex; justify-content: space-between; align-items: center; border-left: 4px solid #dc2626; }
        .bottom-nav { position: fixed; bottom: 0; left: 0; width: 100%; background: white; display: flex; border-top: 1px solid #e5e7eb; box-shadow: 0 -2px 10px rgba(0,0,0,0.05); z-index: 50; }
        .bottom-nav button { flex: 1; padding: 12px 0; font-weight: bold; color: #6b7280; text-align: center; transition: 0.2s; }
        .bottom-nav button.active { color: #dc2626; border-top: 3px solid #dc2626; background: #fef2f2; }
        .hidden-view { display: none; }
        .search-input { width: 100%; padding: 12px 16px; border-radius: 30px; border: 1px solid #d1d5db; font-size: 16px; background: white; box-shadow: inset 0 2px 4px rgba(0,0,0,0.02); }
        .btn-cobrar { background: #10b981; color: white; padding: 16px; border-radius: 16px; font-weight: bold; text-align: center; margin: 10px 15px; }
        .input-field { width: 100%; border: 1px solid #d1d5db; padding: 12px 16px; border-radius: 12px; font-size: 16px; background: white; margin-bottom: 10px; }
        .select-field { width: 100%; border: 1px solid #d1d5db; padding: 12px 16px; border-radius: 12px; font-size: 16px; background: white; margin-bottom: 10px; }
        
        /* Estilos para el lector de código de barras */
        #reader { width: 100%; max-width: 400px; margin: 0 auto; background: #000; border-radius: 16px; overflow: hidden; }
        #scanner-modal { background: rgba(0,0,0,0.8); z-index: 999; }
        .input-group { display: flex; gap: 10px; align-items: center; }
        .input-group .input-field { flex: 1; margin-bottom: 0; }
        .btn-scan { background: #1f2937; color: white; width: 50px; height: 50px; border-radius: 12px; display: flex; align-items: center; justify-content: center; flex-shrink: 0; }
        
        /* Estilo del bloque amarillo de configuración */
        .config-box { background: #fffbeb; border: 1px solid #fde68a; border-radius: 16px; padding: 16px; margin-bottom: 15px; }
        .config-box .title { font-size: 12px; font-weight: 800; color: #92400e; text-transform: uppercase; border-bottom: 1px solid #fde68a; padding-bottom: 8px; margin-bottom: 12px; display: flex; align-items: center; gap: 8px; }
        .config-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }
        .config-grid .full { grid-column: span 2; }
        .config-label { font-size: 11px; font-weight: 800; color: #92400e; text-transform: uppercase; display: block; margin-bottom: 4px; }
        .config-input { width: 100%; border: 1px solid #fde68a; padding: 10px; border-radius: 10px; font-weight: bold; text-align: center; background: white; }
        .config-input.price { text-align: right; padding-right: 30px; }
        .price-wrapper { position: relative; }
        .price-wrapper span { position: absolute; left: 12px; top: 50%; transform: translateY(-50%); font-weight: bold; color: #92400e; }
    </style>
</head>
<body>

    <!-- HEADER -->
    <div class="header">🛒 Mini Super Carlitos</div>

    <!-- VISTA CAJA (PREDETERMINADA) -->
    <div id="view-caja" class="p-4">
        <div class="flex gap-2 mb-4 items-center">
            <button onclick="openScannerMobile('pos')" class="bg-gray-800 text-white w-12 h-12 rounded-full flex items-center justify-center shadow-md shrink-0">
                <i class="fas fa-camera text-lg"></i>
            </button>
            <input type="text" id="search-mobile" oninput="searchMobile(this.value)" placeholder="🔍 Buscar producto..." class="search-input flex-1">
        </div>

        <!-- GRID DE PRODUCTOS -->
        <div id="product-list-mobile" class="space-y-2"></div>

        <!-- CARRITO -->
        <div style="background: white; border-radius: 16px; padding: 16px; margin-top: 20px; box-shadow: 0 -4px 10px rgba(0,0,0,0.05);">
            <div class="flex justify-between items-center mb-2">
                <span class="font-bold text-lg">🛒 Carrito</span>
                <button onclick="clearCartMobile()" class="text-red-500 text-sm font-bold">Vaciar</button>
            </div>
            <div id="cart-list-mobile" class="mb-4 min-h-[60px]">
                <p class="text-gray-400 text-center text-sm">Agrega productos aquí</p>
            </div>
            <div class="flex justify-between font-black text-xl mb-2 border-t pt-2">
                <span>Total</span>
                <span id="total-mobile" class="text-red-600">$0.00</span>
            </div>
            <button onclick="openCheckoutMobile()" class="btn-cobrar w-full">
                <i class="fas fa-money-bill-wave"></i> Cobrar en Efectivo
            </button>
        </div>
    </div>

    <!-- VISTA REGISTRO DE PRODUCTO -->
    <div id="view-registro" class="hidden-view p-4">
        <div class="bg-white rounded-2xl p-4 shadow-sm mb-4">
            <h3 class="font-bold text-lg mb-3">📦 Registrar Producto</h3>
            
            <!-- CAMPO DE CÓDIGO CON BOTÓN DE CÁMARA -->
            <div class="input-group mb-3">
                <input type="text" id="p-code-mob" placeholder="Código de Barras" class="input-field">
                <button onclick="openScannerMobile('register')" class="btn-scan shadow-sm">
                    <i class="fas fa-camera text-lg"></i>
                </button>
            </div>

            <input type="text" id="p-name-mob" required placeholder="Nombre del producto" class="input-field">
            <input type="number" id="p-cost-mob" placeholder="Costo ($)" class="input-field">
            
            <select id="p-cat-mob" class="select-field">
                <option>Refrescos</option>
                <option>Sabritas</option>
                <option>Galletas</option>
                <option>Cigarros</option>
                <option>Otros</option>
            </select>
            
            <select id="p-type-mob" onchange="toggleConfigMobile()" class="select-field">
                <option value="pieza">Por Pieza</option>
                <option value="granel">Por kilos</option>
                <option value="cajapieza">Caja / Pieza</option>
                <option value="cajetilla">Cajetilla / Cigarro</option>
            </select>

            <!-- BLOQUE DE CONFIGURACIÓN DINÁMICO (El cuadro amarillo) -->
            <div id="mixed-fields-mob" class="hidden config-box">
                <div class="title"><i class="fas fa-boxes"></i> Configuración de <span id="mixed-label-title-mob">Paquete</span></div>
                
                <div id="mixed-price-fields-mob" class="config-grid">
                    <!-- Precio Caja / Cajetilla -->
                    <div class="price-wrapper">
                        <span>$</span>
                        <input type="number" id="p-price-mob" step="0.01" placeholder="0.00" class="config-input price">
                    </div>
                    <!-- Precio Pieza / Cigarro -->
                    <div class="price-wrapper">
                        <span>$</span>
                        <input type="number" id="p-piece-price-mob" step="0.01" placeholder="0.00" class="config-input price">
                    </div>
                    <!-- Piezas por Caja / Cigarros x Cajetilla -->
                    <div class="full">
                        <label class="config-label"><span id="mixed-label-per-box-mob">Piezas x Caja</span></label>
                        <input type="number" id="p-pieces-per-box-mob" min="1" placeholder="24" value="24" class="config-input">
                    </div>
                </div>

                <div class="config-grid mt-3 border-t border-amber-200 pt-3">
                    <div>
                        <label class="config-label"><span id="mixed-label-stock-boxes-mob">Cajas</span> en Stock</label>
                        <input type="number" id="p-stock-boxes-mob" min="0" placeholder="0" class="config-input">
                    </div>
                    <div>
                        <label class="config-label"><span id="mixed-label-loose-mob">Piezas</span> Sueltas</label>
                        <input type="number" id="p-stock-loose-mob" min="0" placeholder="0" class="config-input">
                    </div>
                </div>
            </div>

            <input type="number" id="p-price-mob" required placeholder="Precio Venta ($)" class="input-field">
            <input type="number" id="p-stock-mob" placeholder="Stock (piezas)" class="input-field">
            
            <button onclick="saveProductMobile()" class="w-full bg-blue-600 text-white p-4 rounded-xl font-bold shadow-lg">
                <i class="fas fa-save"></i> Guardar Producto
            </button>
        </div>
    </div>

    <!-- MODAL DEL ESCÁNER -->
    <div id="scanner-modal" class="hidden-view fixed inset-0 flex items-center justify-center p-4">
        <div class="bg-white rounded-3xl w-full max-w-md p-4 shadow-2xl relative">
            <button onclick="closeScannerMobile()" class="absolute top-2 right-4 text-red-500 font-bold text-xl">✕</button>
            <h3 class="text-center font-bold mb-3 text-gray-700">Escanear Código</h3>
            <div id="reader"></div>
        </div>
    </div>

    <!-- NAVEGADOR INFERIOR -->
    <div class="bottom-nav">
        <button id="btn-caja" class="active" onclick="switchView('caja')"><i class="fas fa-cash-register block text-xl mb-1"></i> Caja</button>
        <button id="btn-registro" onclick="switchView('registro')"><i class="fas fa-plus-circle block text-xl mb-1"></i> Registrar</button>
    </div>

    <script>
        // 🔥 CONFIGURACIÓN DE FIREBASE
        const firebaseConfig = {
            apiKey: "AIzaSyA1mBLI8HYE5vrjtC0yiGgij2zvocfckac",
            authDomain: "tiendacarlitos-a5b1d.firebaseapp.com",
            databaseURL: "https://tiendacarlitos-a5b1d-default-rtdb.firebaseio.com",
            projectId: "tiendacarlitos-a5b1d",
            storageBucket: "tiendacarlitos-a5b1d.firebasestorage.app",
            messagingSenderId: "653941208919",
            appId: "1:653941208919:web:42bb4150ccc3f558f53012",
            measurementId: "G-RKZJMETTW7"
        };
        
        // Inicializar Firebase
        firebase.initializeApp(firebaseConfig);
        const dbRef = firebase.database().ref('miniSuperDb');

        let products = [];
        let cart = [];
        let html5QrCode = null;

        // 🔄 CONEXIÓN EN TIEMPO REAL A FIREBASE
        dbRef.on('value', (snapshot) => {
            const data = snapshot.val();
            if (data && data.products) {
                products = data.products;
                // 👇 CORREGIDO: Ya NO ejecutamos searchMobile aquí
                renderCartMobile();
            }
        });

        function switchView(view) {
            document.getElementById('view-caja').classList.toggle('hidden-view', view !== 'caja');
            document.getElementById('view-registro').classList.toggle('hidden-view', view !== 'registro');
            document.getElementById('btn-caja').classList.toggle('active', view === 'caja');
            document.getElementById('btn-registro').classList.toggle('active', view === 'registro');
        }

        // 🔍 BÚSQUEDA
        function searchMobile(query) {
            const q = query.toLowerCase().trim();
            const container = document.getElementById('product-list-mobile');
            
            // 👇 CORREGIDO: Si está vacío, limpiamos el contenedor y NO mostramos nada
            if (q === '') {
                container.innerHTML = '';
                return;
            }

            const filtered = products.filter(p => p.name.toLowerCase().includes(q) || (p.code && p.code.toLowerCase().includes(q)));
            
            if(filtered.length === 0) {
                container.innerHTML = '<p class="text-center text-gray-400 py-10">No encontrado</p>';
                return;
            }
            
            container.innerHTML = filtered.map(p => `
                <div onclick="addToCartMobile(${p.id})" class="product-card cursor-pointer active:scale-95 transition-transform">
                    <div>
                        <span class="font-bold text-sm block">${p.name}</span>
                        <span class="text-xs text-gray-500">Stock: ${p.stock}</span>
                    </div>
                    <span class="font-black text-red-600">$${Number(p.price).toFixed(2)}</span>
                </div>
            `).join('');
        }

        // 🛒 AGREGAR AL CARRITO
        function addToCartMobile(id) {
            const product = products.find(p => p.id === id);
            if (!product || product.stock <= 0) return Swal.fire('Sin stock', 'No hay unidades', 'warning');

            if (product.category === 'Cigarros' && (product.type === 'mixto' || product.type === 'cajetilla')) {
                const piecePrice = product.piecePrice || 0;
                const boxPrice = product.price || 0;
                const ppb = product.piecesPerBox || 20;

                Swal.fire({
                    title: product.name,
                    html: `¿Cajetilla ($${boxPrice}) o Cigarro ($${piecePrice})?`,
                    showDenyButton: true, showCancelButton: true,
                    confirmButtonText: `📦 Cajetilla`,
                    denyButtonText: `🚬 Cigarro`,
                    cancelButtonText: 'Cancelar'
                }).then(res => {
                    if (res.isConfirmed) {
                        const exist = cart.find(c => c.id === product.id && c.sellType === 'Cajetilla');
                        if(exist) { exist.qty++; } else { cart.push({ ...product, qty: 1, price: boxPrice, sellType: 'Cajetilla', name: `${product.name} (Cajetilla)` }); }
                        renderCartMobile();
                    } else if (res.isDenied) {
                        const exist = cart.find(c => c.id === product.id && c.sellType === 'Suelto');
                        if(exist) { exist.qty++; } else { cart.push({ ...product, qty: 1, price: piecePrice, sellType: 'Suelto', name: `${product.name} (Suelto)` }); }
                        renderCartMobile();
                    }
                });
                return;
            }

            const exist = cart.find(c => c.id === id && !c.sellType);
            if(exist) { exist.qty++; } else { cart.push({ ...product, qty: 1 }); }
            renderCartMobile();
        }

        function renderCartMobile() {
            const container = document.getElementById('cart-list-mobile');
            const totalSpan = document.getElementById('total-mobile');
            let total = 0;

            if(cart.length === 0) {
                container.innerHTML = '<p class="text-gray-400 text-center text-sm">Agrega productos aquí</p>';
                totalSpan.innerText = '$0.00';
                return;
            }

            container.innerHTML = cart.map(item => {
                const sub = item.price * item.qty;
                total += sub;
                return `
                    <div class="cart-item">
                        <div>
                            <div class="font-bold text-sm">${item.name}</div>
                            <div class="flex items-center gap-2 mt-1">
                                <button onclick="changeQtyMobile('${item.id}', -1, '${item.sellType || ''}')" class="bg-gray-200 w-6 h-6 rounded flex items-center justify-center text-xs">-</button>
                                <span class="w-6 text-center font-bold text-xs">${item.qty}</span>
                                <button onclick="changeQtyMobile('${item.id}', 1, '${item.sellType || ''}')" class="bg-gray-200 w-6 h-6 rounded flex items-center justify-center text-xs">+</button>
                            </div>
                        </div>
                        <div class="text-right">
                            <span class="font-black text-red-600 text-sm">$${sub.toFixed(2)}</span>
                            <br><button onclick="removeItemMobile('${item.id}', '${item.sellType || ''}')" class="text-red-400 text-[10px]">Eliminar</button>
                        </div>
                    </div>
                `;
            }).join('');
            totalSpan.innerText = `$${total.toFixed(2)}`;
        }

        function changeQtyMobile(id, delta, sellType) {
            const item = cart.find(c => c.id == id && (c.sellType || '') === sellType);
            if(!item) return;
            if(delta < 0 && item.qty <= 1) return removeItemMobile(id, sellType);
            item.qty += delta;
            renderCartMobile();
        }

        function removeItemMobile(id, sellType) {
            cart = cart.filter(c => !(c.id == id && (c.sellType || '') === sellType));
            renderCartMobile();
        }

        function clearCartMobile() { cart = []; renderCartMobile(); }

        // 💵 COBRAR
        function openCheckoutMobile() {
            if(cart.length === 0) return Swal.fire('Carrito vacío', 'Agrega productos', 'warning');
            const total = cart.reduce((acc, i) => acc + (i.price * i.qty), 0);
            
            Swal.fire({
                title: `Total: $${total.toFixed(2)}`,
                input: 'number',
                inputLabel: 'Efectivo recibido',
                inputPlaceholder: '$',
                showCancelButton: true,
                confirmButtonText: 'Confirmar',
                preConfirm: (received) => {
                    if(parseFloat(received) < total) return Swal.showValidationMessage('Falta dinero');
                    return parseFloat(received);
                }
            }).then(res => {
                if(res.isConfirmed) {
                    const change = res.value - total;
                    Swal.fire(`✅ Venta completada`, `Cambio: $${change.toFixed(2)}`, 'success');
                    finalizeSaleMobile(total);
                }
            });
        }

        function finalizeSaleMobile(total) {
            dbRef.once('value').then(snap => {
                const data = snap.val() || { products: [], dailySales: 0, salesHistory: [] };
                
                cart.forEach(item => {
                    const prod = data.products.find(p => p.id === item.id);
                    if(prod) prod.stock -= item.qty;
                });

                data.dailySales = (data.dailySales || 0) + total;
                if(!data.salesHistory) data.salesHistory = [];
                data.salesHistory.push({ date: new Date().toISOString(), total, items: cart, seller: 'Móvil' });

                dbRef.set(data);
                cart = [];
                renderCartMobile();
            });
        }

        // 📦 REGISTRAR PRODUCTO (CON MULTIPLICACIÓN Y TIPOS)
        function saveProductMobile() {
            const name = document.getElementById('p-name-mob').value.trim();
            if(!name) return Swal.fire('Error', 'El nombre es obligatorio', 'warning');
            
            const type = document.getElementById('p-type-mob').value;
            
            let stockVal = 0;
            
            // Si es caja o cajetilla, calculamos el stock multiplicando
            if (type === 'cajapieza' || type === 'cajetilla') {
                const boxes = parseFloat(document.getElementById('p-stock-boxes-mob').value) || 0;
                const ppb = parseInt(document.getElementById('p-pieces-per-box-mob').value) || 1;
                const loose = parseFloat(document.getElementById('p-stock-loose-mob').value) || 0;
                
                stockVal = (boxes * ppb) + loose; 
            } else {
                // Para venta por pieza normal o por kilos
                stockVal = parseFloat(document.getElementById('p-stock-mob').value) || 0;
            }

            const newProduct = {
                id: Date.now(),
                code: document.getElementById('p-code-mob').value.trim() || '',
                name: name.toUpperCase(),
                category: document.getElementById('p-cat-mob').value,
                type: type,
                cost: parseFloat(document.getElementById('p-cost-mob').value) || 0,
                stock: stockVal,
                expiryDate: null
            };

            // Guardar los precios y configuraciones específicas según el tipo
            if (type === 'cajetilla' || (type === 'cajapieza' && document.getElementById('p-cat-mob').value === 'Cigarros')) {
                // Para Cajetilla / Cigarro
                newProduct.price = parseFloat(document.getElementById('p-price-mob').value) || 0;
                newProduct.piecePrice = parseFloat(document.getElementById('p-piece-price-mob').value) || 0;
                newProduct.piecesPerBox = parseInt(document.getElementById('p-pieces-per-box-mob').value) || 20;
            } else if (type === 'cajapieza') {
                // Para Caja / Pieza (Solo usamos el precio unitario estándar)
                newProduct.price = parseFloat(document.getElementById('p-price-mob').value) || 0;
                newProduct.piecePrice = 0;
                newProduct.piecesPerBox = parseInt(document.getElementById('p-pieces-per-box-mob').value) || 24;
            } else {
                // Por Pieza / Por kilos
                newProduct.price = parseFloat(document.getElementById('p-price-mob').value) || 0;
                newProduct.piecePrice = null;
                newProduct.piecesPerBox = null;
            }

            dbRef.once('value').then(snap => {
                const data = snap.val() || { products: [] };
                data.products.push(newProduct);
                dbRef.set(data);
                
                Swal.fire('✅ Guardado', 'Producto registrado en la nube', 'success');
                document.getElementById('p-name-mob').value = '';
                document.getElementById('p-code-mob').value = '';
                document.getElementById('p-cost-mob').value = '';
                document.getElementById('p-price-mob').value = '';
                document.getElementById('p-stock-mob').value = '';
                document.getElementById('p-stock-boxes-mob').value = '';
                document.getElementById('p-stock-loose-mob').value = '';
            });
        }

        // 👇 FUNCIÓN PARA MOSTRAR/OCULTAR EL CUADRO AMARILLO (CON CAMBIO DE TEXTOS)
        function toggleConfigMobile() {
            const type = document.getElementById('p-type-mob').value;
            const mixedFields = document.getElementById('mixed-fields-mob');
            const cat = document.getElementById('p-cat-mob').value;

            if (type === 'cajapieza' || type === 'cajetilla') {
                mixedFields.classList.remove('hidden');
                
                // Cambiar textos dinámicamente
                const title = document.getElementById('mixed-label-title-mob');
                const labelPriceBox = document.querySelector('#mixed-price-fields-mob div:nth-child(1) .config-label');
                const labelPiece = document.querySelector('#mixed-price-fields-mob div:nth-child(2) .config-label');
                const labelPerBox = document.getElementById('mixed-label-per-box-mob');
                const labelStockBoxes = document.getElementById('mixed-label-stock-boxes-mob');
                const labelLoose = document.getElementById('mixed-label-loose-mob');

                if (type === 'cajetilla' || (type === 'cajapieza' && cat === 'Cigarros')) {
                    // Cajetilla / Cigarro
                    if(title) title.innerText = 'Cajetilla';
                    if(labelPriceBox) labelPriceBox.innerText = 'Precio x Cajetilla';
                    if(labelPiece) labelPiece.innerText = 'Precio x Cigarro';
                    if(labelPerBox) labelPerBox.innerText = 'Cigarros x Cajetilla';
                    if(labelStockBoxes) labelStockBoxes.innerText = 'Cajetillas';
                    if(labelLoose) labelLoose.innerText = 'Cigarros Sueltos';
                    
                    document.getElementById('p-price-mob').parentElement.parentElement.style.display = 'grid';
                    document.getElementById('p-piece-price-mob').parentElement.parentElement.style.display = 'grid';
                } else {
                    // Caja / Pieza
                    if(title) title.innerText = 'Paquete';
                    if(labelPriceBox) labelPriceBox.innerText = 'Precio x Caja';
                    if(labelPiece) labelPiece.innerText = 'Precio x Pieza';
                    if(labelPerBox) labelPerBox.innerText = 'Piezas x Caja';
                    if(labelStockBoxes) labelStockBoxes.innerText = 'Cajas';
                    if(labelLoose) labelLoose.innerText = 'Piezas Sueltas';
                    
                    // Ocultar precios en Caja/Pieza
                    document.getElementById('p-price-mob').parentElement.parentElement.style.display = 'none';
                    document.getElementById('p-piece-price-mob').parentElement.parentElement.style.display = 'none';
                }
            } else {
                mixedFields.classList.add('hidden');
            }
        }

        // 👇 FUNCIONES DE LA CÁMARA
        function openScannerMobile(context) {
            const modal = document.getElementById('scanner-modal');
            if (!modal) return;

            modal.classList.remove('hidden-view');

            if (html5QrCode) {
                html5QrCode.stop().catch(err => console.error(err));
            }
            
            html5QrCode = new Html5Qrcode("reader");
            html5QrCode.start(
                { facingMode: "environment" },
                { fps: 10, qrbox: { width: 250, height: 250 } },
                (decodedText) => {
                    closeScannerMobile();
                    
                    if (context === 'pos') {
                        document.getElementById('search-mobile').value = decodedText;
                        searchMobile(decodedText);
                    } else if (context === 'register') {
                        document.getElementById('p-code-mob').value = decodedText;
                    }
                    
                    Swal.fire({
                        icon: 'success',
                        title: 'Código escaneado',
                        text: decodedText,
                        timer: 1500,
                        showConfirmButton: false
                    });
                },
                (errorMessage) => {}
            ).catch(err => {
                Swal.fire('Error', 'No se pudo acceder a la cámara. Asegúrate de dar permisos.', 'error');
                closeScannerMobile();
            });
        }

        function closeScannerMobile() {
            const modal = document.getElementById('scanner-modal');
            if (modal) modal.classList.add('hidden-view');
            if (html5QrCode) {
                html5QrCode.stop().then(() => html5QrCode.clear()).catch(err => console.error(err));
            }
        }
    </script>
</body>
</html>
