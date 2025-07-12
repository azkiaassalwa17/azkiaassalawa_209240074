document.addEventListener('DOMContentLoaded', function () {
    const cartBtn = document.querySelector('.cart-btn');
    const cartSidebar = document.querySelector('.cart-sidebar');
    const overlay = document.querySelector('.overlay');
    const closeCartBtn = document.querySelector('.close-cart');
    const cartItemsContainer = document.querySelector('.cart-items');
    const emptyCartMsg = document.querySelector('.empty-cart-message');
    const totalAmountEl = document.querySelector('.total-amount');
    const cartCount = document.querySelector('.cart-count');
    const addToCartBtns = document.querySelectorAll('.add-to-cart');
    const checkoutBtn = document.querySelector('.checkout-btn');

    let cart = [];

    cartBtn.addEventListener('click', () => {
        cartSidebar.classList.add('open');
        overlay.classList.add('active');
    });

    function hideCart() {
        cartSidebar.classList.remove('open');
        overlay.classList.remove('active');
    }

    closeCartBtn.addEventListener('click', hideCart);
    overlay.addEventListener('click', hideCart);

    function addToCart(productId) {
        const productCard = document.querySelector(`.product-card[data-id="${productId}"]`);
        const title = productCard.querySelector('.product-title').textContent;
        const price = parseInt(productCard.querySelector('.product-price').textContent.replace(/\D/g, ''));
        const image = productCard.querySelector('.product-image').src;

        const existingItem = cart.find(item => item.id === productId);

        if (existingItem) {
            existingItem.quantity += 1;
        } else {
            cart.push({
                id: productId,
                title,
                price,
                image,
                quantity: 1
            });
        }

        updateCart();
        showFeedback(`${title} ditambahkan ke keranjang`);
    }

    function updateCart() {
        cartItemsContainer.innerHTML = '';

        if (cart.length === 0) {
            cartItemsContainer.innerHTML = '<p class="empty-cart-message">Keranjang belanja kosong</p>';
            totalAmountEl.textContent = '0';
            cartCount.textContent = '0';
            return;
        }

        let totalAmount = 0;
        let totalItems = 0;

        cart.forEach(item => {
            totalAmount += item.price * item.quantity;
            totalItems += item.quantity;

            const cartItemEl = document.createElement('div');
            cartItemEl.className = 'cart-item';
            cartItemEl.dataset.id = item.id;
            cartItemEl.innerHTML = `
                <img src="${item.image}" class="cart-item-img">
                <div class="cart-item-details">
                    <h4 class="cart-item-title">${item.title}</h4>
                    <p class="cart-item-price">Rp ${item.price.toLocaleString('id-ID')}</p>
                    <div class="cart-item-actions">
                        <button class="quantity-btn minus">-</button>
                        <span class="cart-item-quantity">${item.quantity}</span>
                        <button class="quantity-btn plus">+</button>
                        <span class="remove-item">Hapus</span>
                    </div>
                </div>
            `;

            cartItemsContainer.appendChild(cartItemEl);
        });

        totalAmountEl.textContent = totalAmount.toLocaleString('id-ID');
        cartCount.textContent = totalItems;

        document.querySelectorAll('.quantity-btn.minus').forEach(btn => btn.addEventListener('click', decreaseQuantity));
        document.querySelectorAll('.quantity-btn.plus').forEach(btn => btn.addEventListener('click', increaseQuantity));
        document.querySelectorAll('.remove-item').forEach(btn => btn.addEventListener('click', removeItem));
    }

    function decreaseQuantity(e) {
        const itemId = parseInt(e.target.closest('.cart-item').dataset.id);
        const item = cart.find(i => i.id === itemId);
        if (item.quantity > 1) {
            item.quantity -= 1;
        } else {
            cart = cart.filter(i => i.id !== itemId);
        }
        updateCart();
    }

    function increaseQuantity(e) {
        const itemId = parseInt(e.target.closest('.cart-item').dataset.id);
        const item = cart.find(i => i.id === itemId);
        item.quantity += 1;
        updateCart();
    }

    function removeItem(e) {
        const itemId = parseInt(e.target.closest('.cart-item').dataset.id);
        cart = cart.filter(i => i.id !== itemId);
        updateCart();
    }

    checkoutBtn.addEventListener('click', function () {
        if (cart.length === 0) {
            showFeedback('Keranjang belanja kosong');
        } else {
            showFeedback('Pesanan berhasil diproses');
            cart = [];
            updateCart();
            hideCart();
        }
    });

    function showFeedback(message) {
        const feedback = document.createElement('div');
        feedback.className = 'feedback-message';
        feedback.textContent = message;
        Object.assign(feedback.style, {
            position: 'fixed',
            bottom: '20px',
            left: '50%',
            transform: 'translateX(-50%)',
            backgroundColor: '#333',
            color: 'white',
            padding: '10px 20px',
            borderRadius: '5px',
            zIndex: '1000'
        });
        document.body.appendChild(feedback);

        setTimeout(() => {
            feedback.style.opacity = '0';
            feedback.style.transition = 'opacity 0.5s ease';
            setTimeout(() => feedback.remove(), 500);
        }, 2000);
    }

    addToCartBtns.forEach(btn => {
        btn.addEventListener('click', function () {
            const productId = parseInt(this.closest('.product-card').dataset.id);
            addToCart(productId);
        });
    });
});
