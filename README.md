import React, { useState, useEffect, useMemo } from 'react';
import { 
  ShoppingBag, 
  Coffee, 
  Utensils, 
  Cake, 
  Search, 
  Plus, 
  Minus, 
  Trash2, 
  CreditCard, 
  DollarSign, 
  QrCode, 
  TrendingUp, 
  Package, 
  Users, 
  History, 
  X, 
  CheckCircle, 
  AlertTriangle, 
  Clock, 
  Receipt, 
  PauseCircle,
  Play,
  Edit,
  ArrowUpRight
} from 'lucide-react';

// --- INITIAL MOCK DATA ---
const INITIAL_CATEGORIES = [
  { id: 'all', name: 'ทั้งหมด (All)' },
  { id: 'tea', name: 'ชาตรา (Signature Tea)' },
  { id: 'coffee', name: 'กาแฟโบราณ/สด (Coffee)' },
  { id: 'milk', name: 'นมสด & โกโก้ (Milk & Cocoa)' },
  { id: 'toppings', name: 'ท็อปปิ้ง (Toppings)' },
  { id: 'snacks', name: 'ทานเล่น (Snacks)' },
];

const INITIAL_PRODUCTS = [
  { id: 'p1', name: 'ชาไทยโบราณ (Thai Milk Tea)', category: 'tea', price: 45, stock: 60, image: '🧋', options: { size: true, sweetness: true, toppings: true } },
  { id: 'p2', name: 'ชาเขียวมะลิ (Green Tea Jasmine)', category: 'tea', price: 45, stock: 40, image: '🍵', options: { size: true, sweetness: true, toppings: true } },
  { id: 'p3', name: 'ชาดำเย็น (Black Tea)', category: 'tea', price: 35, stock: 50, image: '🧋', options: { size: true, sweetness: true, toppings: true } },
  { id: 'p4', name: 'โอเลี้ยงโบราณ (O-Lieng)', category: 'coffee', price: 35, stock: 35, image: '☕', options: { size: true, sweetness: true, toppings: false } },
  { id: 'p5', name: 'โกโก้เข้มข้น (Rich Cocoa)', category: 'milk', price: 50, stock: 25, image: '🍫', options: { size: true, sweetness: true, toppings: true } },
  { id: 'p6', name: 'นมสดคาราเมล (Fresh Milk Caramel)', category: 'milk', price: 45, stock: 18, image: '🥛', options: { size: true, sweetness: true, toppings: true } },
  { id: 'p7', name: 'ไข่มุกบราวน์ซูการ์ (Brown Sugar Pearl)', category: 'toppings', price: 10, stock: 100, image: '🟤', options: { size: false, sweetness: false, toppings: false } },
  { id: 'p8', name: 'เยลลี่บراун (Brown Jelly)', category: 'toppings', price: 10, stock: 80, image: '🍮', options: { size: false, sweetness: false, toppings: false } },
  { id: 'p9', name: 'ขนมปังปิ้งสังขยา (Toast with Pandan Custard)', category: 'snacks', price: 30, stock: 15, image: '🍞', options: { size: false, sweetness: false, toppings: false } },
  { id: 'p10', name: 'เฟรนช์ฟรายส์ทอด (French Fries)', category: 'snacks', price: 45, stock: 20, image: '🍟', options: { size: false, sweetness: false, toppings: false } },
];

const TOPPINGS_LIST = [
  { id: 't1', name: 'ไข่มุก (+10฿)', price: 10 },
  { id: 't2', name: 'เยลลี่บราวน์ซูการ์ (+10฿)', price: 10 },
  { id: 't3', name: 'วิปครีม (+15฿)', price: 15 },
  { id: 't4', name: 'ชีสพ่นไฟ (+20฿)', price: 20 },
];

export default function App() {
  // --- STATE ---
  const [role, setRole] = useState('cashier'); // 'cashier' | 'manager'
  const [products, setProducts] = useState(INITIAL_PRODUCTS);
  const [activeCategory, setActiveCategory] = useState('all');
  const [searchQuery, setSearchQuery] = useState('');
  
  // Cart & Order state
  const [cart, setCart] = useState([]);
  const [discountPercent, setDiscountPercent] = useState(0);
  const [orderNote, setOrderNote] = useState('');
  const [heldOrders, setHeldOrders] = useState([]);
  const [orderHistory, setOrderHistory] = useState([
    {
      id: 'ORD-1001',
      timestamp: new Date(Date.now() - 3600000 * 2).toISOString(),
      items: [{ name: 'ชาไทยโบราณ', quantity: 2, price: 45, options: ['ไซส์: M', 'ความหวาน: 100%'] }],
      subtotal: 90,
      discount: 0,
      tax: 6.3,
      total: 96.3,
      paymentMethod: 'cash',
      cashier: 'สมชาย (Cashier 1)',
      note: 'หวานปกติ แยกน้ำแข็ง'
    },
    {
      id: 'ORD-1002',
      timestamp: new Date(Date.now() - 3600000 * 1).toISOString(),
      items: [{ name: 'โกโก้เข้มข้น', quantity: 1, price: 50, options: ['ไซส์: L', 'ความหวาน: 50%'] }],
      subtotal: 60,
      discount: 0,
      tax: 4.2,
      total: 64.2,
      paymentMethod: 'promptpay',
      cashier: 'สมชาย (Cashier 1)',
      note: ''
    }
  ]);

  // Modals & UI States
  const [selectedProductForOpt, setSelectedProductForOpt] = useState(null);
  const [showPaymentModal, setShowPaymentModal] = useState(false);
  const [showShiftModal, setShowShiftModal] = useState(false);
  const [showReceiptModal, setShowReceiptModal] = useState(null);
  const [managerTab, setManagerTab] = useState('dashboard'); // 'dashboard' | 'inventory' | 'history' | 'staff'

  // Shift Register State
  const [shiftStatus, setShiftStatus] = useState({
    isOpen: true,
    openedAt: new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' }),
    initialFloat: 1500,
    cashSales: 96.3,
    promptPaySales: 64.2,
  });

  // Customize Item Temp State
  const [customOptions, setCustomOptions] = useState({
    size: 'M',
    sizePrice: 0,
    sweetness: '100%',
    selectedToppings: []
  });

  // Filtered Products
  const filteredProducts = useMemo(() => {
    return products.filter(p => {
      const matchesCategory = activeCategory === 'all' || p.category === activeCategory;
      const matchesSearch = p.name.toLowerCase().includes(searchQuery.toLowerCase());
      return matchesCategory && matchesSearch;
    });
  }, [products, activeCategory, searchQuery]);

  // --- CART CALCULATIONS ---
  const subtotal = useMemo(() => {
    return cart.reduce((sum, item) => sum + (item.unitPrice * item.quantity), 0);
  }, [cart]);

  const discountAmount = useMemo(() => {
    return (subtotal * discountPercent) / 100;
  }, [subtotal, discountPercent]);

  const taxAmount = useMemo(() => {
    return ((subtotal - discountAmount) * 0.07); // 7% VAT
  }, [subtotal, discountAmount]);

  const grandTotal = useMemo(() => {
    return subtotal - discountAmount + taxAmount;
  }, [subtotal, discountAmount, taxAmount]);

  // --- HANDLERS ---
  const handleProductClick = (product) => {
    if (product.stock <= 0) return;

    if (product.options.size || product.options.sweetness || product.options.toppings) {
      setSelectedProductForOpt(product);
      setCustomOptions({
        size: 'M',
        sizePrice: 0,
        sweetness: '100%',
        selectedToppings: []
      });
    } else {
      addToCartDirect(product, 1, product.price, []);
    }
  };

  const addToCartDirect = (product, qty, unitPrice, optionsSummary = []) => {
    const existingIndex = cart.findIndex(item => 
      item.productId === product.id && JSON.stringify(item.optionsSummary) === JSON.stringify(optionsSummary)
    );

    if (existingIndex > -1) {
      const newCart = [...cart];
      newCart[existingIndex].quantity += qty;
      setCart(newCart);
    } else {
      setCart([...cart, {
        cartItemId: Date.now().toString() + Math.random(),
        productId: product.id,
        name: product.name,
        unitPrice: unitPrice,
        quantity: qty,
        optionsSummary: optionsSummary
      }]);
    }
  };

  const confirmCustomOptions = () => {
    if (!selectedProductForOpt) return;
    
    let extra = customOptions.sizePrice;
    const toppingsListText = customOptions.selectedToppings.map(t => {
      extra += t.price;
      return t.name;
    });

    const optionsSummary = [];
    if (selectedProductForOpt.options.size) optionsSummary.push(`ไซส์: ${customOptions.size}`);
    if (selectedProductForOpt.options.sweetness) optionsSummary.push(`ความหวาน: ${customOptions.sweetness}`);
    if (toppingsListText.length > 0) optionsSummary.push(`ท็อปปิ้ง: ${toppingsListText.join(', ')}`);

    const finalUnitPrice = selectedProductForOpt.price + extra;

    addToCartDirect(selectedProductForOpt, 1, finalUnitPrice, optionsSummary);
    setSelectedProductForOpt(null);
  };

  const updateCartQty = (cartItemId, delta) => {
    setCart(prev => prev.map(item => {
      if (item.cartItemId === cartItemId) {
        const newQty = item.quantity + delta;
        return newQty > 0 ? { ...item, quantity: newQty } : null;
      }
      return item;
    }).filter(Boolean));
  };

  const handleHoldOrder = () => {
    if (cart.length === 0) return;
    setHeldOrders([...heldOrders, { id: `HOLD-${Date.now().toString().slice(-4)}`, cart, note: orderNote, timestamp: new Date().toLocaleTimeString() }]);
    setCart([]);
    setOrderNote('');
  };

  const handleRestoreHoldOrder = (held) => {
    setCart(held.cart);
    setOrderNote(held.note || '');
    setHeldOrders(heldOrders.filter(h => h.id !== held.id));
  };

  const handleCompletePayment = (paymentMethod, cashTendered = 0) => {
    const newOrder = {
      id: `ORD-${Math.floor(1000 + Math.random() * 9000)}`,
      timestamp: new Date().toISOString(),
      items: cart.map(i => ({ name: i.name, quantity: i.quantity, price: i.unitPrice, options: i.optionsSummary })),
      subtotal,
      discount: discountAmount,
      tax: taxAmount,
      total: grandTotal,
      paymentMethod,
      cashTendered: paymentMethod === 'cash' ? cashTendered : grandTotal,
      change: paymentMethod === 'cash' ? cashTendered - grandTotal : 0,
      cashier: 'สมชาย (Cashier 1)',
      note: orderNote
    };

    // Deduct stock
    setProducts(prev => prev.map(p => {
      const cartItem = cart.find(ci => ci.productId === p.id);
      if (cartItem) {
        return { ...p, stock: Math.max(0, p.stock - cartItem.quantity) };
      }
      return p;
    }));

    // Update Shift Stats
    setShiftStatus(prev => ({
      ...prev,
      cashSales: paymentMethod === 'cash' ? prev.cashSales + grandTotal : prev.cashSales,
      promptPaySales: paymentMethod === 'promptpay' ? prev.promptPaySales + grandTotal : prev.promptPaySales
    }));

    setOrderHistory([newOrder, ...orderHistory]);
    setShowPaymentModal(false);
    setShowReceiptModal(newOrder);
    setCart([]);
    setDiscountPercent(0);
    setOrderNote('');
  };

  return (
    <div className="flex flex-col h-screen bg-slate-100 font-sans text-slate-800 antialiased overflow-hidden">
      
      {/* --- TOP HEADER --- */}
      <header className="bg-amber-800 text-white px-6 py-3 flex items-center justify-between shadow-md z-10">
        <div className="flex items-center gap-3">
          <div className="p-2 bg-amber-900 rounded-xl shadow border border-amber-700">
            <Coffee className="w-6 h-6 text-amber-200" />
          </div>
          <div>
            <h1 className="font-bold text-lg leading-tight">CHAREE CHA & POS</h1>
            <p className="text-xs text-amber-200">ระบบขายหน้าร้านและบริหารจัดการร้านชา</p>
          </div>
        </div>

        {/* Shift Quick Status */}
        <div className="hidden md:flex items-center gap-5 bg-amber-900/60 px-4 py-1.5 rounded-full border border-amber-700 text-xs">
          <div className="flex items-center gap-2">
            <Clock className="w-4 h-4 text-emerald-300" />
            <span>เปิดกะ: {shiftStatus.openedAt} น.</span>
          </div>
          <div className="h-3 w-[1px] bg-amber-700"></div>
          <div>
            ยอดกะนี้: <span className="text-emerald-300 font-bold">฿{(shiftStatus.cashSales + shiftStatus.promptPaySales).toLocaleString()}</span>
          </div>
          <button 
            onClick={() => setShowShiftModal(true)} 
            className="ml-2 text-amber-200 hover:text-white underline font-medium"
          >
            จัดการกะ
          </button>
        </div>

        {/* Role Switcher */}
        <div className="flex items-center gap-2 bg-amber-900 p-1 rounded-xl border border-amber-700">
          <button
            onClick={() => setRole('cashier')}
            className={`flex items-center gap-2 px-4 py-1.5 rounded-lg text-sm font-semibold transition-all ${
              role === 'cashier' 
                ? 'bg-amber-600 text-white shadow-md' 
                : 'text-amber-200 hover:text-white'
            }`}
          >
            <ShoppingBag className="w-4 h-4" />
            <span>แคชเชียร์</span>
          </button>

          <button
            onClick={() => setRole('manager')}
            className={`flex items-center gap-2 px-4 py-1.5 rounded-lg text-sm font-semibold transition-all ${
              role === 'manager' 
                ? 'bg-amber-600 text-white shadow-md' 
                : 'text-amber-200 hover:text-white'
            }`}
          >
            <TrendingUp className="w-4 h-4" />
            <span>ผู้จัดการ</span>
          </button>
        </div>
      </header>

      {/* --- MAIN BODY --- */}
      {role === 'cashier' ? (
        <div className="flex flex-1 overflow-hidden">
          
          {/* LEFT: PRODUCTS GRID */}
          <div className="flex-1 flex flex-col min-w-0 bg-slate-50 border-r border-slate-200">
            
            {/* Search & Categories */}
            <div className="p-4 bg-white border-b border-slate-200 shadow-sm space-y-3">
              <div className="relative">
                <Search className="absolute left-3 top-1/2 -translate-y-1/2 w-5 h-5 text-slate-400" />
                <input
                  type="text"
                  placeholder="ค้นหาชื่อเครื่องดื่มหรือสินค้า..."
                  value={searchQuery}
                  onChange={(e) => setSearchQuery(e.target.value)}
                  className="w-full pl-10 pr-4 py-2 bg-slate-100 rounded-xl border-none focus:ring-2 focus:ring-amber-500 text-sm font-medium outline-none"
                />
                {searchQuery && (
                  <button onClick={() => setSearchQuery('')} className="absolute right-3 top-1/2 -translate-y-1/2 text-slate-400 hover:text-slate-600">
                    <X className="w-4 h-4" />
                  </button>
                )}
              </div>

              <div className="flex items-center gap-2 overflow-x-auto pb-1 scrollbar-none">
                {INITIAL_CATEGORIES.map((cat) => (
                  <button
                    key={cat.id}
                    onClick={() => setActiveCategory(cat.id)}
                    className={`px-4 py-2 rounded-xl text-xs font-bold whitespace-nowrap transition-all border ${
                      activeCategory === cat.id
                        ? 'bg-amber-700 text-white border-amber-700 shadow-sm'
                        : 'bg-slate-100 text-slate-600 border-slate-200 hover:bg-slate-200'
                    }`}
                  >
                    {cat.name}
                  </button>
                ))}
              </div>
            </div>

            {/* Products Grid */}
            <div className="flex-1 overflow-y-auto p-4">
              <div className="grid grid-cols-2 sm:grid-cols-3 lg:grid-cols-4 gap-4">
                {filteredProducts.map((product) => {
                  const isLowStock = product.stock > 0 && product.stock <= 5;
                  const isOutOfStock = product.stock <= 0;

                  return (
                    <div
                      key={product.id}
                      onClick={() => !isOutOfStock && handleProductClick(product)}
                      className={`relative bg-white rounded-2xl p-4 border border-slate-200 flex flex-col justify-between transition-all cursor-pointer hover:shadow-md hover:border-amber-400 group ${
                        isOutOfStock ? 'opacity-50 cursor-not-allowed bg-slate-100' : ''
                      }`}
                    >
                      <div>
                        <div className="w-full h-24 bg-amber-50 rounded-xl flex items-center justify-center text-4xl mb-3 group-hover:scale-105 transition-transform">
                          {product.image}
                        </div>
                        <h3 className="font-bold text-slate-800 text-sm line-clamp-2 leading-snug mb-1">
                          {product.name}
                        </h3>
                        {(product.options.size || product.options.sweetness) && (
                          <span className="inline-block bg-slate-100 text-slate-500 text-[10px] px-2 py-0.5 rounded-md font-medium mb-2">
                            ปรับแต่งรสชาติได้
                          </span>
                        )}
                      </div>

                      <div className="flex items-center justify-between mt-2 pt-2 border-t border-slate-100">
                        <span className="font-extrabold text-amber-700 text-base">
                          ฿{product.price}
                        </span>
                        <div>
                          {isOutOfStock ? (
                            <span className="text-[10px] bg-red-100 text-red-600 font-bold px-2 py-0.5 rounded-full">หมด</span>
                          ) : isLowStock ? (
                            <span className="text-[10px] bg-amber-100 text-amber-700 font-bold px-2 py-0.5 rounded-full flex items-center gap-0.5">
                              <AlertTriangle className="w-3 h-3" /> เหลือ {product.stock}
                            </span>
                          ) : (
                            <span className="text-[10px] bg-slate-100 text-slate-500 font-medium px-2 py-0.5 rounded-full">
                              สต็อก {product.stock}
                            </span>
                          )}
                        </div>
                      </div>
                    </div>
                  );
                })}
              </div>

              {filteredProducts.length === 0 && (
                <div className="flex flex-col items-center justify-center h-64 text-slate-400">
                  <Package className="w-12 h-12 mb-2 stroke-1" />
                  <p className="text-sm font-medium">ไม่พบสินค้าในหมวดหมู่นี้</p>
                </div>
              )}
            </div>
          </div>

          {/* RIGHT: CART / ORDER SUMMARY PANEL */}
          <div className="w-96 bg-white flex flex-col shadow-xl z-10 border-l border-slate-200">
            <div className="p-4 border-b border-slate-200 flex items-center justify-between">
              <div className="flex items-center gap-2">
                <Receipt className="w-5 h-5 text-amber-700" />
                <h2 className="font-bold text-slate-800">รายการสั่งซื้อ</h2>
                <span className="bg-amber-100 text-amber-800 text-xs font-bold px-2 py-0.5 rounded-full">
                  {cart.reduce((sum, item) => sum + item.quantity, 0)}
                </span>
              </div>
              
              {cart.length > 0 && (
                <button
                  onClick={() => { setCart([]); setOrderNote(''); }}
                  className="text-xs text-red-500 hover:text-red-700 font-semibold flex items-center gap-1"
                >
                  <Trash2 className="w-3.5 h-3.5" /> ล้าง
                </button>
              )}
            </div>

            {/* Held Orders Bar */}
            {heldOrders.length > 0 && (
              <div className="bg-amber-50 p-2 border-b border-amber-200 flex items-center gap-2 overflow-x-auto">
                <span className="text-xs font-bold text-amber-800 flex items-center gap-1 whitespace-nowrap">
                  <PauseCircle className="w-3.5 h-3.5" /> คิวพัก:
                </span>
                {heldOrders.map((h) => (
                  <button
                    key={h.id}
                    onClick={() => handleRestoreHoldOrder(h)}
                    className="bg-white border border-amber-300 text-amber-900 text-xs px-2 py-1 rounded-lg font-medium hover:bg-amber-100 whitespace-nowrap flex items-center gap-1 shadow-sm"
                  >
                    <span>{h.id}</span>
                    <Play className="w-3 h-3 text-amber-600 fill-amber-600" />
                  </button>
                ))}
              </div>
            )}

            {/* Cart List */}
            <div className="flex-1 overflow-y-auto p-4 divide-y divide-slate-100">
              {cart.length === 0 ? (
                <div className="flex flex-col items-center justify-center h-full text-slate-400 text-center">
                  <ShoppingBag className="w-16 h-16 stroke-1 mb-2 text-slate-300" />
                  <p className="font-medium text-sm">ยังไม่มีรายการสินค้า</p>
                  <p className="text-xs text-slate-400 mt-1">คลิกเลือกเมนูด้านซ้ายเพื่อเริ่มต้นออเดอร์</p>
                </div>
              ) : (
                cart.map((item) => (
                  <div key={item.cartItemId} className="py-3 first:pt-0 last:pb-0">
                    <div className="flex justify-between items-start mb-1">
                      <div>
                        <h4 className="font-bold text-slate-800 text-sm">{item.name}</h4>
                        {item.optionsSummary && item.optionsSummary.length > 0 && (
                          <p className="text-[11px] text-slate-500 leading-tight">
                            {item.optionsSummary.join(' | ')}
                          </p>
                        )}
                      </div>
                      <span className="font-bold text-slate-800 text-sm">
                        ฿{(item.unitPrice * item.quantity).toLocaleString()}
                      </span>
                    </div>

                    <div className="flex items-center justify-between mt-2">
                      <span className="text-xs text-slate-400">@ ฿{item.unitPrice}</span>
                      <div className="flex items-center gap-2 bg-slate-100 rounded-lg p-0.5">
                        <button
                          onClick={() => updateCartQty(item.cartItemId, -1)}
                          className="w-6 h-6 bg-white rounded-md flex items-center justify-center shadow-sm text-slate-600"
                        >
                          <Minus className="w-3 h-3" />
                        </button>
                        <span className="w-6 text-center text-xs font-bold text-slate-800">
                          {item.quantity}
                        </span>
                        <button
                          onClick={() => updateCartQty(item.cartItemId, 1)}
                          className="w-6 h-6 bg-white rounded-md flex items-center justify-center shadow-sm text-slate-600"
                        >
                          <Plus className="w-3 h-3" />
                        </button>
                      </div>
                    </div>
                  </div>
                ))
              )}
            </div>

            {/* Cart Calculations & Note */}
            <div className="p-4 bg-slate-50 border-t border-slate-200 space-y-3">
              <div>
                <input
                  type="text"
                  placeholder="หมายเหตุออเดอร์ (เช่น หวานน้อยพิเศษ, ใส่ถุงแยก)"
                  value={orderNote}
                  onChange={(e) => setOrderNote(e.target.value)}
                  className="w-full px-3 py-1.5 bg-white rounded-xl border border-slate-200 text-xs focus:ring-1 focus:ring-amber-500 outline-none"
                />
              </div>

              <div className="flex items-center justify-between text-xs">
                <span className="text-slate-500 font-medium">ส่วนลด (%)</span>
                <div className="flex gap-1">
                  {[0, 5, 10, 20].map((d) => (
                    <button
                      key={d}
                      onClick={() => setDiscountPercent(d)}
                      className={`px-2.5 py-1 rounded-md border text-[11px] font-bold ${
                        discountPercent === d
                          ? 'bg-amber-700 text-white border-amber-700'
                          : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-100'
                      }`}
                    >
                      {d}%
                    </button>
                  ))}
                </div>
              </div>

              <div className="space-y-1.5 text-xs text-slate-600">
                <div className="flex justify-between">
                  <span>ยอดรวม (Subtotal)</span>
                  <span>฿{subtotal.toFixed(2)}</span>
                </div>
                {discountAmount > 0 && (
                  <div className="flex justify-between text-red-500 font-medium">
                    <span>ส่วนลด ({discountPercent}%)</span>
                    <span>-฿{discountAmount.toFixed(2)}</span>
                  </div>
                )}
                <div className="flex justify-between text-slate-400">
                  <span>ภาษีมูลค่าเพิ่ม (VAT 7%)</span>
                  <span>฿{taxAmount.toFixed(2)}</span>
                </div>
                <div className="flex justify-between items-baseline pt-2 border-t border-slate-200 text-slate-900">
                  <span className="font-bold text-base">ยอดชำระสุทธิ</span>
                  <span className="font-extrabold text-2xl text-amber-700">
                    ฿{grandTotal.toFixed(2)}
                  </span>
                </div>
              </div>

              <div className="grid grid-cols-4 gap-2 pt-1">
                <button
                  disabled={cart.length === 0}
                  onClick={handleHoldOrder}
                  className="col-span-1 bg-amber-100 text-amber-900 hover:bg-amber-200 disabled:opacity-40 font-bold py-3 rounded-xl flex flex-col items-center justify-center gap-1 text-xs"
                >
                  <PauseCircle className="w-4 h-4" />
                  <span>พักบิล</span>
                </button>

                <button
                  disabled={cart.length === 0}
                  onClick={() => setShowPaymentModal(true)}
                  className="col-span-3 bg-amber-700 hover:bg-amber-800 text-white font-bold py-3 rounded-xl shadow-lg disabled:opacity-40 flex items-center justify-center gap-2 text-base transition-all"
                >
                  <CreditCard className="w-5 h-5" />
                  <span>ชำระเงิน (฿{grandTotal.toFixed(2)})</span>
                </button>
              </div>

            </div>
          </div>

        </div>
      ) : (
        
        /* --- MANAGER MODE --- */
        <div className="flex flex-1 overflow-hidden bg-slate-100">
          
          <aside className="w-64 bg-white border-r border-slate-200 p-4 flex flex-col gap-2">
            <div className="px-3 py-2 text-xs font-bold text-slate-400 uppercase tracking-wider">
              ผู้จัดการระบบ
            </div>

            <button
              onClick={() => setManagerTab('dashboard')}
              className={`flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold transition-all ${
                managerTab === 'dashboard' ? 'bg-amber-50 text-amber-800' : 'text-slate-600 hover:bg-slate-50'
              }`}
            >
              <TrendingUp className="w-5 h-5" />
              <span>ภาพรวมยอดขาย</span>
            </button>

            <button
              onClick={() => setManagerTab('inventory')}
              className={`flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold transition-all ${
                managerTab === 'inventory' ? 'bg-amber-50 text-amber-800' : 'text-slate-600 hover:bg-slate-50'
              }`}
            >
              <Package className="w-5 h-5" />
              <span>จัดการสต็อกสินค้า</span>
            </button>

            <button
              onClick={() => setManagerTab('history')}
              className={`flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold transition-all ${
                managerTab === 'history' ? 'bg-amber-50 text-amber-800' : 'text-slate-600 hover:bg-slate-50'
              }`}
            >
              <History className="w-5 h-5" />
              <span>ประวัติคำสั่งซื้อ</span>
            </button>

            <button
              onClick={() => setManagerTab('staff')}
              className={`flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-bold transition-all ${
                managerTab === 'staff' ? 'bg-amber-50 text-amber-800' : 'text-slate-600 hover:bg-slate-50'
              }`}
            >
              <Users className="w-5 h-5" />
              <span>พนักงาน & กะทำงาน</span>
            </button>
          </aside>

          <main className="flex-1 overflow-y-auto p-8">
            
            {managerTab === 'dashboard' && (
              <div className="space-y-6 max-w-6xl mx-auto">
                <h2 className="text-2xl font-black text-slate-800">แดชบอร์ดสรุปยอดขาย (Analytics)</h2>
                
                <div className="grid grid-cols-1 md:grid-cols-4 gap-4">
                  <div className="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
                    <span className="text-xs font-bold text-slate-400">ยอดขายรวมวันนี้</span>
                    <p className="text-2xl font-black text-slate-800 mt-1">
                      ฿{orderHistory.reduce((s, o) => s + o.total, 0).toLocaleString(undefined, { minimumFractionDigits: 2 })}
                    </p>
                    <span className="text-xs text-emerald-600 font-bold mt-2 inline-block flex items-center gap-1">
                      <ArrowUpRight className="w-3.5 h-3.5" /> เติบโตจากเมื่อวาน 12%
                    </span>
                  </div>

                  <div className="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
                    <span className="text-xs font-bold text-slate-400">จำนวนบิลทั้งหมด</span>
                    <p className="text-2xl font-black text-slate-800 mt-1">{orderHistory.length} ออเดอร์</p>
                    <span className="text-xs text-slate-400 mt-2 inline-block">เฉลี่ย ฿{(orderHistory.reduce((s, o) => s + o.total, 0) / (orderHistory.length || 1)).toFixed(0)} ต่อบิล</span>
                  </div>

                  <div className="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
                    <span className="text-xs font-bold text-slate-400">ยอดชำระด้วยเงินสด</span>
                    <p className="text-2xl font-black text-slate-800 mt-1">
                      ฿{orderHistory.filter(o => o.paymentMethod === 'cash').reduce((s, o) => s + o.total, 0).toLocaleString()}
                    </p>
                  </div>

                  <div className="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm">
                    <span className="text-xs font-bold text-slate-400">ยอดชำระ PromptPay</span>
                    <p className="text-2xl font-black text-amber-700 mt-1">
                      ฿{orderHistory.filter(o => o.paymentMethod === 'promptpay').reduce((s, o) => s + o.total, 0).toLocaleString()}
                    </p>
                  </div>
                </div>

                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  <div className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm">
                    <h3 className="font-bold text-slate-800 mb-4 flex items-center gap-2">
                      <Coffee className="w-5 h-5 text-amber-700" /> เมนูขายดีประจำวัน
                    </h3>
                    <div className="space-y-4">
                      {products.slice(0, 4).map((p, idx) => (
                        <div key={p.id} className="flex items-center justify-between">
                          <div className="flex items-center gap-3">
                            <span className="w-6 h-6 rounded-full bg-slate-100 text-slate-600 font-extrabold text-xs flex items-center justify-center">
                              {idx + 1}
                            </span>
                            <div>
                              <p className="font-bold text-slate-800 text-sm">{p.name}</p>
                              <p className="text-xs text-slate-400">ราคา ฿{p.price}</p>
                            </div>
                          </div>
                          <span className="font-bold text-sm text-slate-700">{45 - idx * 8} แก้ว</span>
                        </div>
                      ))}
                    </div>
                  </div>

                  <div className="bg-white p-6 rounded-2xl border border-slate-200 shadow-sm">
                    <h3 className="font-bold text-slate-800 mb-4 flex items-center gap-2">
                      <AlertTriangle className="w-5 h-5 text-amber-500" /> สินค้าใกล้หมดสต็อก
                    </h3>
                    <div className="space-y-3">
                      {products.filter(p => p.stock <= 20).map((p) => (
                        <div key={p.id} className="flex items-center justify-between p-3 bg-amber-50 rounded-xl border border-amber-100">
                          <div className="flex items-center gap-2">
                            <span className="text-xl">{p.image}</span>
                            <span className="font-bold text-amber-900 text-sm">{p.name}</span>
                          </div>
                          <span className="text-xs font-bold bg-amber-200 text-amber-900 px-2.5 py-1 rounded-md">
                            เหลือ {p.stock} ชิ้น
                          </span>
                        </div>
                      ))}
                    </div>
                  </div>
                </div>
              </div>
            )}

            {managerTab === 'inventory' && (
              <div className="space-y-6 max-w-6xl mx-auto">
                <div className="flex justify-between items-center">
                  <h2 className="text-2xl font-black text-slate-800">จัดการสต็อกและสินค้า</h2>
                  <button className="bg-amber-700 hover:bg-amber-800 text-white font-bold px-4 py-2 rounded-xl text-sm flex items-center gap-2">
                    <Plus className="w-4 h-4" /> เพิ่มสินค้าใหม่
                  </button>
                </div>

                <div className="bg-white rounded-2xl border border-slate-200 overflow-hidden shadow-sm">
                  <table className="w-full text-left text-sm">
                    <thead className="bg-slate-50 border-b border-slate-200 text-slate-400 font-bold text-xs uppercase">
                      <tr>
                        <th className="p-4">สินค้า</th>
                        <th className="p-4">หมวดหมู่</th>
                        <th className="p-4">ราคา (บาท)</th>
                        <th className="p-4">คงเหลือ</th>
                        <th className="p-4">สถานะ</th>
                        <th className="p-4 text-right">การจัดการ</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-100">
                      {products.map((p) => (
                        <tr key={p.id} className="hover:bg-slate-50">
                          <td className="p-4 font-bold text-slate-800 flex items-center gap-3">
                            <span className="text-2xl">{p.image}</span>
                            <span>{p.name}</span>
                          </td>
                          <td className="p-4 text-slate-500 font-medium">{p.category}</td>
                          <td className="p-4 font-bold text-slate-800">฿{p.price}</td>
                          <td className="p-4">
                            <div className="flex items-center gap-2">
                              <button 
                                onClick={() => setProducts(products.map(prod => prod.id === p.id ? { ...prod, stock: Math.max(0, prod.stock - 1) } : prod))}
                                className="w-6 h-6 bg-slate-100 rounded flex items-center justify-center font-bold text-slate-600 hover:bg-slate-200"
                              >-</button>
                              <span className="font-bold w-8 text-center">{p.stock}</span>
                              <button 
                                onClick={() => setProducts(products.map(prod => prod.id === p.id ? { ...prod, stock: prod.stock + 1 } : prod))}
                                className="w-6 h-6 bg-slate-100 rounded flex items-center justify-center font-bold text-slate-600 hover:bg-slate-200"
                              >+</button>
                            </div>
                          </td>
                          <td className="p-4">
                            {p.stock <= 0 ? (
                              <span className="bg-red-100 text-red-700 text-xs font-bold px-2.5 py-1 rounded-full">สินค้าหมด</span>
                            ) : p.stock <= 15 ? (
                              <span className="bg-amber-100 text-amber-700 text-xs font-bold px-2.5 py-1 rounded-full">ใกล้หมด</span>
                            ) : (
                              <span className="bg-emerald-100 text-emerald-700 text-xs font-bold px-2.5 py-1 rounded-full">ปกติ</span>
                            )}
                          </td>
                          <td className="p-4 text-right">
                            <button className="text-amber-700 hover:text-amber-900 font-bold text-xs p-2">
                              <Edit className="w-4 h-4 inline" /> แก้ไข
                            </button>
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            )}

            {managerTab === 'history' && (
              <div className="space-y-6 max-w-6xl mx-auto">
                <h2 className="text-2xl font-black text-slate-800">ประวัติคำสั่งซื้อทั้งหมด</h2>

                <div className="bg-white rounded-2xl border border-slate-200 overflow-hidden shadow-sm">
                  <table className="w-full text-left text-sm">
                    <thead className="bg-slate-50 border-b border-slate-200 text-slate-400 font-bold text-xs uppercase">
                      <tr>
                        <th className="p-4">เลขที่บิล</th>
                        <th className="p-4">เวลา</th>
                        <th className="p-4">รายการสินค้า</th>
                        <th className="p-4">ช่องทางชำระ</th>
                        <th className="p-4">ยอดสุทธิ</th>
                        <th className="p-4 text-right">ใบเสร็จ</th>
                      </tr>
                    </thead>
                    <tbody className="divide-y divide-slate-100">
                      {orderHistory.map((ord) => (
                        <tr key={ord.id} className="hover:bg-slate-50">
                          <td className="p-4 font-bold text-slate-800">{ord.id}</td>
                          <td className="p-4 text-slate-500 text-xs">
                            {new Date(ord.timestamp).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })} น.
                          </td>
                          <td className="p-4 text-slate-600">
                            {ord.items.map(i => `${i.name} (x${i.quantity})`).join(', ')}
                          </td>
                          <td className="p-4">
                            <span className={`text-xs font-bold px-2 py-1 rounded-md ${
                              ord.paymentMethod === 'cash' ? 'bg-slate-100 text-slate-700' : 'bg-amber-100 text-amber-800'
                            }`}>
                              {ord.paymentMethod === 'cash' ? 'เงินสด' : 'PromptPay QR'}
                            </span>
                          </td>
                          <td className="p-4 font-black text-amber-700">฿{ord.total.toFixed(2)}</td>
                          <td className="p-4 text-right">
                            <button
                              onClick={() => setShowReceiptModal(ord)}
                              className="text-xs bg-slate-100 hover:bg-slate-200 text-slate-700 font-bold px-3 py-1.5 rounded-lg border border-slate-200"
                            >
                              ดูใบเสร็จ
                            </button>
                          </td>
                        </tr>
                      ))}
                    </tbody>
                  </table>
                </div>
              </div>
            )}

            {managerTab === 'staff' && (
              <div className="space-y-6 max-w-6xl mx-auto">
                <h2 className="text-2xl font-black text-slate-800">พนักงานและกะการทำงาน</h2>
                <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
                  <div className="bg-white p-5 rounded-2xl border border-slate-200 shadow-sm flex items-center gap-4">
                    <div className="w-12 h-12 bg-amber-100 rounded-full flex items-center justify-center text-amber-800 font-bold text-lg">
                      สช
                    </div>
                    <div>
                      <h3 className="font-bold text-slate-800">สมชาย (Cashier 1)</h3>
                      <p className="text-xs text-slate-400">สถานะกะ: กำลังปฏิบัติงาน</p>
                      <span className="text-[10px] bg-emerald-100 text-emerald-700 px-2 py-0.5 rounded-full font-bold mt-1 inline-block">Online</span>
                    </div>
                  </div>
                </div>
              </div>
            )}

          </main>
        </div>
      )}

      {/* --- MODAL 1: CUSTOMIZATION MODAL --- */}
      {selectedProductForOpt && (
        <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-3xl max-w-md w-full overflow-hidden shadow-2xl">
            
            <div className="p-6 bg-amber-800 text-white flex justify-between items-center">
              <div className="flex items-center gap-3">
                <span className="text-3xl">{selectedProductForOpt.image}</span>
                <div>
                  <h3 className="font-bold text-lg">{selectedProductForOpt.name}</h3>
                  <p className="text-xs text-amber-200">ราคาเริ่มต้น ฿{selectedProductForOpt.price}</p>
                </div>
              </div>
              <button onClick={() => setSelectedProductForOpt(null)} className="text-amber-200 hover:text-white">
                <X className="w-6 h-6" />
              </button>
            </div>

            <div className="p-6 space-y-5 max-h-[60vh] overflow-y-auto">
              {selectedProductForOpt.options.size && (
                <div>
                  <label className="text-xs font-bold text-slate-400 uppercase tracking-wider block mb-2">ขนาดแก้ว (Size)</label>
                  <div className="grid grid-cols-3 gap-2">
                    {[
                      { name: 'S', price: -5, label: 'S (-5฿)' },
                      { name: 'M', price: 0, label: 'M (ปกติ)' },
                      { name: 'L', price: 10, label: 'L (+10฿)' }
                    ].map((sz) => (
                      <button
                        key={sz.name}
                        onClick={() => setCustomOptions({ ...customOptions, size: sz.name, sizePrice: sz.price })}
                        className={`p-2.5 rounded-xl text-xs font-bold border transition-all ${
                          customOptions.size === sz.name
                            ? 'bg-amber-700 text-white border-amber-700 shadow-sm'
                            : 'bg-slate-50 text-slate-700 border-slate-200 hover:bg-slate-100'
                        }`}
                      >
                        {sz.label}
                      </button>
                    ))}
                  </div>
                </div>
              )}

              {selectedProductForOpt.options.sweetness && (
                <div>
                  <label className="text-xs font-bold text-slate-400 uppercase tracking-wider block mb-2">ระดับความหวาน (Sweetness)</label>
                  <div className="grid grid-cols-4 gap-2">
                    {['0%', '25%', '50%', '100%'].map((sw) => (
                      <button
                        key={sw}
                        onClick={() => setCustomOptions({ ...customOptions, sweetness: sw })}
                        className={`p-2 rounded-xl text-xs font-bold border transition-all ${
                          customOptions.sweetness === sw
                            ? 'bg-amber-700 text-white border-amber-700 shadow-sm'
                            : 'bg-slate-50 text-slate-700 border-slate-200 hover:bg-slate-100'
                        }`}
                      >
                        {sw}
                      </button>
                    ))}
                  </div>
                </div>
              )}

              {selectedProductForOpt.options.toppings && (
                <div>
                  <label className="text-xs font-bold text-slate-400 uppercase tracking-wider block mb-2">เพิ่มท็อปปิ้ง (Toppings)</label>
                  <div className="space-y-2">
                    {TOPPINGS_LIST.map((t) => {
                      const isChecked = customOptions.selectedToppings.some(item => item.id === t.id);
                      return (
                        <div
                          key={t.id}
                          onClick={() => {
                            if (isChecked) {
                              setCustomOptions({
                                ...customOptions,
                                selectedToppings: customOptions.selectedToppings.filter(item => item.id !== t.id)
                              });
                            } else {
                              setCustomOptions({
                                ...customOptions,
                                selectedToppings: [...customOptions.selectedToppings, t]
                              });
                            }
                          }}
                          className={`p-3 rounded-xl border flex items-center justify-between cursor-pointer text-xs font-bold transition-all ${
                            isChecked ? 'bg-amber-50 border-amber-300 text-amber-900' : 'bg-slate-50 border-slate-200 text-slate-700'
                          }`}
                        >
                          <span>{t.name}</span>
                          <span className="text-amber-700">เลือกแล้ว</span>
                        </div>
                      );
                    })}
                  </div>
                </div>
              )}
            </div>

            <div className="p-4 bg-slate-50 border-t border-slate-200 flex gap-3">
              <button
                onClick={() => setSelectedProductForOpt(null)}
                className="flex-1 bg-white border border-slate-200 text-slate-600 font-bold py-3 rounded-xl text-sm hover:bg-slate-100"
              >
                ยกเลิก
              </button>
              <button
                onClick={confirmCustomOptions}
                className="flex-1 bg-amber-700 hover:bg-amber-800 text-white font-bold py-3 rounded-xl text-sm shadow-md"
              >
                ยืนยันใส่ตะกร้า
              </button>
            </div>

          </div>
        </div>
      )}

      {/* --- MODAL 2: PAYMENT MODAL --- */}
      {showPaymentModal && (
        <PaymentModal
          totalAmount={grandTotal}
          onClose={() => setShowPaymentModal(false)}
          onConfirmPayment={handleCompletePayment}
        />
      )}

      {/* --- MODAL 3: SHIFT MODAL --- */}
      {showShiftModal && (
        <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-3xl max-w-md w-full p-6 shadow-2xl space-y-5">
            <div className="flex justify-between items-center border-b border-slate-100 pb-3">
              <h3 className="font-bold text-lg text-slate-800 flex items-center gap-2">
                <Clock className="w-5 h-5 text-amber-700" /> จัดการกะเงินสด
              </h3>
              <button onClick={() => setShowShiftModal(false)} className="text-slate-400 hover:text-slate-600">
                <X className="w-5 h-5" />
              </button>
            </div>

            <div className="space-y-3 text-sm">
              <div className="bg-slate-50 p-3 rounded-xl flex justify-between">
                <span className="text-slate-500">เงินทอนตั้งต้น</span>
                <span className="font-bold text-slate-800">฿{shiftStatus.initialFloat.toLocaleString()}</span>
              </div>
              <div className="bg-slate-50 p-3 rounded-xl flex justify-between">
                <span className="text-slate-500">ยอดขายเงินสด</span>
                <span className="font-bold text-emerald-600">+฿{shiftStatus.cashSales.toFixed(2)}</span>
              </div>
              <div className="bg-slate-50 p-3 rounded-xl flex justify-between">
                <span className="text-slate-500">ยอดขาย PromptPay</span>
                <span className="font-bold text-amber-700">+฿{shiftStatus.promptPaySales.toFixed(2)}</span>
              </div>
              <div className="border-t border-slate-200 pt-3 flex justify-between font-bold text-base">
                <span>เงินสดรวมในลิ้นชัก</span>
                <span className="text-amber-700">
                  ฿{(shiftStatus.initialFloat + shiftStatus.cashSales).toFixed(2)}
                </span>
              </div>
            </div>

            <button 
              onClick={() => { alert('ปิดกะและบันทึกยอดสำเร็จ'); setShowShiftModal(false); }}
              className="w-full bg-red-600 hover:bg-red-700 text-white font-bold py-3 rounded-xl text-sm"
            >
              ปิดกะทำงาน (End Shift)
            </button>
          </div>
        </div>
      )}

      {/* --- MODAL 4: RECEIPT MODAL --- */}
      {showReceiptModal && (
        <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
          <div className="bg-white rounded-3xl max-w-sm w-full p-6 shadow-2xl space-y-4">
            <div className="text-center space-y-1 pb-3 border-b border-dashed border-slate-300">
              <div className="inline-block p-2 bg-amber-50 rounded-full mb-1">
                <CheckCircle className="w-8 h-8 text-emerald-500" />
              </div>
              <h3 className="font-black text-slate-800 text-lg">CHAREE CHA</h3>
              <p className="text-xs text-slate-400">ใบเสร็จรับเงินอย่างย่อ</p>
              <p className="text-[11px] text-slate-400">เลขที่บิล: {showReceiptModal.id}</p>
            </div>

            <div className="space-y-2 text-xs">
              {showReceiptModal.items.map((i, idx) => (
                <div key={idx} className="flex justify-between">
                  <div>
                    <span className="font-bold">{i.name} x{i.quantity}</span>
                    {i.options && <p className="text-[10px] text-slate-400">{i.options.join(', ')}</p>}
                  </div>
                  <span className="font-bold">฿{(i.price * i.quantity).toFixed(2)}</span>
                </div>
              ))}
              {showReceiptModal.note && (
                <p className="text-[11px] text-amber-800 bg-amber-50 p-1.5 rounded">หมายเหตุ: {showReceiptModal.note}</p>
              )}
            </div>

            <div className="border-t border-dashed border-slate-300 pt-3 space-y-1 text-xs">
              <div className="flex justify-between text-slate-500">
                <span>ส่วนลด</span>
                <span>-฿{showReceiptModal.discount.toFixed(2)}</span>
              </div>
              <div className="flex justify-between text-slate-500">
                <span>VAT 7%</span>
                <span>฿{showReceiptModal.tax.toFixed(2)}</span>
              </div>
              <div className="flex justify-between font-extrabold text-sm text-slate-900 pt-1">
                <span>ยอดสุทธิ</span>
                <span>฿{showReceiptModal.total.toFixed(2)}</span>
              </div>
            </div>

            <button
              onClick={() => setShowReceiptModal(null)}
              className="w-full bg-slate-900 hover:bg-slate-800 text-white font-bold py-3 rounded-xl text-sm"
            >
              ปิดหน้าต่าง
            </button>
          </div>
        </div>
      )}

    </div>
  );
}

// --- SUB-COMPONENT: PAYMENT MODAL ---
function PaymentModal({ totalAmount, onClose, onConfirmPayment }) {
  const [method, setMethod] = useState('cash'); // 'cash' | 'promptpay'
  const [cashTendered, setCashTendered] = useState('');

  const numericCash = parseFloat(cashTendered) || 0;
  const changeAmount = numericCash - totalAmount;

  const quickCashOptions = [
    totalAmount,
    Math.ceil(totalAmount / 50) * 50,
    Math.ceil(totalAmount / 100) * 100,
    100,
    500,
    1000
  ].filter((val, index, self) => val >= totalAmount && self.indexOf(val) === index);

  return (
    <div className="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center p-4 z-50">
      <div className="bg-white rounded-3xl max-w-lg w-full overflow-hidden shadow-2xl flex flex-col">
        
        <div className="p-5 bg-amber-800 text-white flex justify-between items-center">
          <h3 className="font-bold text-lg flex items-center gap-2">
            <CreditCard className="w-5 h-5 text-amber-200" /> ชำระเงิน (Checkout)
          </h3>
          <button onClick={onClose} className="text-amber-200 hover:text-white">
            <X className="w-6 h-6" />
          </button>
        </div>

        <div className="p-6 space-y-6 flex-1">
          <div className="text-center bg-slate-50 p-4 rounded-2xl border border-slate-200">
            <span className="text-xs font-bold text-slate-400 uppercase">ยอดที่ต้องชำระสุทธิ</span>
            <div className="text-3xl font-black text-amber-700 mt-1">
              ฿{totalAmount.toFixed(2)}
            </div>
          </div>

          <div className="grid grid-cols-2 gap-3">
            <button
              onClick={() => setMethod('cash')}
              className={`p-3 rounded-2xl border flex items-center justify-center gap-2 font-bold text-sm transition-all ${
                method === 'cash' 
                  ? 'bg-amber-700 text-white border-amber-700 shadow-sm' 
                  : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-50'
              }`}
            >
              <DollarSign className="w-5 h-5" /> เงินสด (Cash)
            </button>

            <button
              onClick={() => setMethod('promptpay')}
              className={`p-3 rounded-2xl border flex items-center justify-center gap-2 font-bold text-sm transition-all ${
                method === 'promptpay' 
                  ? 'bg-amber-700 text-white border-amber-700 shadow-sm' 
                  : 'bg-white text-slate-600 border-slate-200 hover:bg-slate-50'
              }`}
            >
              <QrCode className="w-5 h-5" /> PromptPay QR
            </button>
          </div>

          {method === 'cash' ? (
            <div className="space-y-4">
              <div>
                <label className="text-xs font-bold text-slate-400 block mb-1">รับเงินสดมา (บาท)</label>
                <input
                  type="number"
                  placeholder="0.00"
                  value={cashTendered}
                  onChange={(e) => setCashTendered(e.target.value)}
                  className="w-full text-center text-2xl font-black py-3 bg-slate-100 rounded-xl border-none focus:ring-2 focus:ring-amber-500 outline-none"
                />
              </div>

              <div className="flex flex-wrap gap-2">
                {quickCashOptions.map((amt) => (
                  <button
                    key={amt}
                    onClick={() => setCashTendered(amt.toString())}
                    className="flex-1 py-2 px-3 bg-slate-100 hover:bg-slate-200 rounded-xl text-xs font-bold text-slate-700 border border-slate-200"
                  >
                    ฿{amt}
                  </button>
                ))}
              </div>

              <div className="flex justify-between items-center p-4 bg-emerald-50 rounded-2xl border border-emerald-200 text-emerald-900">
                <span className="font-bold text-sm">เงินทอน (Change):</span>
                <span className="text-2xl font-black">
                  {changeAmount >= 0 ? `฿${changeAmount.toFixed(2)}` : 'รับเงินยังไม่ครบ'}
                </span>
              </div>
            </div>
          ) : (
            <div className="flex flex-col items-center justify-center p-4 bg-slate-50 rounded-2xl border border-slate-200 text-center space-y-3">
              <div className="w-40 h-40 bg-white p-2 rounded-xl border border-slate-300 flex items-center justify-center shadow-inner">
                <svg className="w-full h-full text-slate-800" viewBox="0 0 24 24" fill="currentColor">
                  <path d="M2 2h8v8H2V2zm2 2v4h4V4H4zm9-2h8v8h-8V2zm2 2v4h4V4h-4zM2 14h8v8H2v-8zm2 2v4h4v-4H4zm11-2h2v2h-2v-2zm-2 2h2v2h-2v-2zm4 0h2v2h-2v-2zm-2 2h2v2h-2v-2zm2 2h2v2h-2v-2zm-4 0h2v2h-2v-2z"/>
                </svg>
              </div>
              <p className="text-xs font-bold text-slate-600">สแกน QR Code เพื่อชำระเงินผ่าน Mobile Banking</p>
            </div>
          )}
        </div>

        <div className="p-4 bg-slate-50 border-t border-slate-200 flex gap-3">
          <button
            onClick={onClose}
            className="flex-1 bg-white border border-slate-200 text-slate-600 font-bold py-3 rounded-xl text-sm hover:bg-slate-100"
          >
            ยกเลิก
          </button>
          <button
            disabled={method === 'cash' && changeAmount < 0}
            onClick={() => onConfirmPayment(method, numericCash)}
            className="flex-1 bg-amber-700 hover:bg-amber-800 text-white font-bold py-3 rounded-xl text-sm shadow-md disabled:opacity-40"
          >
            ยืนยันการรับเงิน
          </button>
        </div>

      </div>
    </div>
  );
}
