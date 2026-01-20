import React, { useState, useEffect } from 'react';
import { 
  LayoutDashboard, Users, Package, List, Download, Upload, Settings,
  Menu, FileBarChart2, LogOut, ShieldCheck, Layers, Zap, Bell, Search, Printer, Share2
} from 'lucide-react';
import { 
  Account, Product, StockInOrder, StockOutOrder, InventoryRecord, AppSettings, AccountType, ProductGroup
} from './types';
import { shareAppContent } from './utils/exportUtils';

import Dashboard from './screens/Dashboard';
import AccountsScreen from './screens/AccountsScreen';
import ProductsScreen from './screens/ProductsScreen';
import ProductListScreen from './screens/ProductListScreen';
import StockInScreen from './screens/StockInScreen';
import StockOutScreen from './screens/StockOutScreen';
import InventoryScreen from './screens/InventoryScreen';
import ReportsScreen from './screens/ReportsScreen';
import SettingsScreen from './screens/SettingsScreen';
import GroupsScreen from './screens/GroupsScreen';

export enum Tab {
  DASHBOARD = 'dashboard',
  ACCOUNTS = 'accounts',
  GROUPS = 'groups',
  PRODUCTS = 'products',
  PRODUCT_LIST = 'product_list',
  STOCK_IN = 'stock_in',
  STOCK_OUT = 'stock_out',
  INVENTORY = 'inventory',
  REPORTS = 'reports',
  SETTINGS = 'settings'
}

export type ProductFilterType = 'all' | 'available' | 'out_of_stock' | 'low_stock' | 'near_expiry';

const App: React.FC = () => {
  const [activeTab, setActiveTab] = useState<Tab>(Tab.DASHBOARD);
  const [activeProductFilter, setActiveProductFilter] = useState<ProductFilterType>('all');
  const [isSidebarOpen, setIsSidebarOpen] = useState(false);
  const [isAuthenticated, setIsAuthenticated] = useState(false);
  const [passInput, setPassInput] = useState('');

  // تهيئة الإعدادات الافتراضية مع جلب البيانات من التخزين المحلي
  const [settings, setSettings] = useState<AppSettings>(() => {
    const saved = localStorage.getItem('safwan1_settings');
    return saved ? JSON.parse(saved) : {
      isPasswordEnabled: true,
      password: '1234',
      syncEmail: '',
      expiryWarningDays: 30,
      businessProfile: { 
        name: 'مؤسسة صفوان التجارية', 
        logo: '', 
        header: 'إدارة المخازن الذكية', 
        footer: 'SAFWAN1 Premium v4.0' 
      }
    };
  });

  // إدارة البيانات الأساسية للتطبيق
  const [accounts, setAccounts] = useState<Account[]>(() => JSON.parse(localStorage.getItem('safwan1_accounts') || '[]'));
  const [productGroups, setProductGroups] = useState<ProductGroup[]>(() => JSON.parse(localStorage.getItem('safwan1_groups') || '[]'));
  const [products, setProducts] = useState<Product[]>(() => JSON.parse(localStorage.getItem('safwan1_products') || '[]'));
  const [stockInOrders, setStockInOrders] = useState<StockInOrder[]>(() => JSON.parse(localStorage.getItem('safwan1_stockin') || '[]'));
  const [stockOutOrders, setStockOutOrders] = useState<StockOutOrder[]>(() => JSON.parse(localStorage.getItem('safwan1_stockout') || '[]'));
  const [inventoryRecords, setInventoryRecords] = useState<InventoryRecord[]>(() => JSON.parse(localStorage.getItem('safwan1_inventory') || '[]'));

  // مراقبة وحفظ البيانات تلقائياً عند أي تغيير لضمان عدم الضياع
  useEffect(() => {
    localStorage.setItem('safwan1_accounts', JSON.stringify(accounts));
    localStorage.setItem('safwan1_groups', JSON.stringify(productGroups));
    localStorage.setItem('safwan1_products', JSON.stringify(products));
    localStorage.setItem('safwan1_stockin', JSON.stringify(stockInOrders));
    localStorage.setItem('safwan1_stockout', JSON.stringify(stockOutOrders));
    localStorage.setItem('safwan1_inventory', JSON.stringify(inventoryRecords));
    localStorage.setItem('safwan1_settings', JSON.stringify(settings));
  }, [accounts, productGroups, products, stockInOrders, stockOutOrders, inventoryRecords, settings]);

  const handleLogin = () => {
    if (passInput === settings.password) setIsAuthenticated(true);
    else { 
      setPassInput(''); 
      alert('كلمة المرور خاطئة، يرجى المحاولة مرة أخرى.'); 
    }
  };

  const handleNavigateWithFilter = (filter: ProductFilterType) => {
    setActiveProductFilter(filter);
    setActiveTab(Tab.PRODUCT_LIST);
  };

  // شاشة تسجيل الدخول المحمية
  if (!isAuthenticated && settings.isPasswordEnabled) {
    return (
      <div className="h-screen bg-slate-950 flex items-center justify-center p-6 rtl">
        <div className="bg-slate-900 w-full max-w-sm rounded-[2.5rem] p-10 text-center shadow-2xl z-10 border border-slate-800 animate-in fade-in zoom-in duration-500">
          <div className="mb-8 w-20 h-20 bg-amber-400 rounded-2xl mx-auto flex items-center justify-center text-slate-900 font-black text-2xl shadow-lg border-4 border-white">S</div>
          <h2 className="text-3xl font-black text-white mb-2">SAFWAN<span className="text-amber-400">1</span></h2>
          <p className="text-slate-500 text-[10px] mb-10 uppercase tracking-widest font-bold">نظام الإدارة المخزنية الذكي</p>
          <div className="space-y-4">
            <input 
              type="password" 
              value={passInput} 
              autoFocus
              onChange={e => setPassInput(e.target.value)}
              className="w-full bg-slate-800 border-2 border-slate-700 rounded-2xl py-4 text-center text-2xl text-white outline-none focus:border-amber-400 transition-all font-black tracking-widest"
              placeholder="••••"
              onKeyDown={e => e.key === 'Enter' && handleLogin()}
            />
            <button 
              onClick={handleLogin} 
              className="w-full py-4 rounded-2xl bg-amber-400 text-slate-900 font-black hover:bg-amber-300 transition-all active:scale-95 shadow-xl shadow-amber-400/20"
            >
              دخول آمن
            </button>
          </div>
        </div>
      </div>
    );
  }

  const menuItems = [
    { id: Tab.DASHBOARD, label: 'لوحة التحكم', icon: LayoutDashboard },
    { id: Tab.ACCOUNTS, label: 'الدليل المالي', icon: Users },
    { id: Tab.GROUPS, label: 'التصنيفات', icon: Layers },
    { id: Tab.PRODUCTS, label: 'إضافة أصناف', icon: Package },
    { id: Tab.PRODUCT_LIST, label: 'جرد البضائع', icon: List },
    { id: Tab.STOCK_IN, label: 'أوامر توريد', icon: Download },
    { id: Tab.STOCK_OUT, label: 'أوامر صرف', icon: Upload },
    { id: Tab.INVENTORY, label: 'الجرد الدوري', icon: ShieldCheck },
    { id: Tab.REPORTS, label: 'التقارير', icon: FileBarChart2 },
    { id: Tab.SETTINGS, label: 'الإعدادات', icon: Settings },
  ];

  return (
    <div className="flex h-screen bg-slate-50 rtl text-right overflow-hidden">
      
      {/* القائمة الجانبية السوداء (Sidebar) */}
      <aside className={`fixed lg:static inset-y-0 right-0 z-50 w-64 bg-slate-950 transition-all duration-300 shadow-2xl ${isSidebarOpen ? 'translate-x-0' : 'translate-x-full lg:translate-x-0'} flex flex-col`}>
        <div className="p-8 flex flex-col items-center gap-3 border-b border-white/5">
          <div className="w-12 h-12 bg-amber-400 rounded-xl flex items-center justify-center text-slate-900 font-black text-xl shadow-lg border-2 border-white/20">S</div>
          <h1 className="text-lg font-black text-white tracking-tighter">SAFWAN<span className="text-amber-400">1</span></h1>
        </div>
        
        <nav className="flex-1 mt-4 px-3 space-y-1 overflow-y-auto no-scrollbar">
          {menuItems.map(item => (
            <button 
              key={item.id} 
              onClick={() => { setActiveTab(item.id); setIsSidebarOpen(false); }} 
              className={`w-full flex items-center gap-3 px-4 py-3 rounded-xl transition-all group ${
                activeTab === item.id 
                  ? 'bg-amber-400 text-slate-900 font-black shadow-lg shadow-amber-400/10' 
                  : 'text-slate-400 hover:bg-white/5 hover:text-white'
              }`}
            >
              <item.icon size={18} className={activeTab === item.id ? 'text-slate-900' : 'text-slate-400 group-hover:text-white'} /> 
              <span className="text-sm font-bold">{item.label}</span>
            </button>
          ))}
        </nav>
        
        <div className="p-4 border-t border-white/5">
            <button 
              onClick={() => setIsAuthenticated(false)} 
              className="w-full flex items-center justify-center gap-2 py-3 rounded-xl text-slate-500 hover:text-red-400 hover:bg-red-400/10 transition-all text-xs font-bold"
            >
                <LogOut size={16} /> خروج آمن من النظام
            </button>
        </div>
      </aside>

      {/* المحتوى الرئيسي (Main Content) */}
      <main className="flex-1 flex flex-col min-w-0 bg-transparent relative">
        <header className="h-16 bg-white border-b border-slate-200 px-6 flex items-center justify-between z-40 shadow-sm">
          <div className="flex items-center gap-3">
            <button className="lg:hidden p-2 bg-slate-100 rounded-lg text-slate-600" onClick={() => setIsSidebarOpen(true)}>
              <Menu size={20} />
            </button>
            <h2 className="text-lg font-black text-slate-800">{menuItems.find(i => i.id === activeTab)?.label}</h2>
          </div>
          <div className="flex items-center gap-2">
            <button onClick={() => window.print()} className="p-2 text-slate-400 hover:text-slate-900 transition-all" title="طباعة الصفحة">
              <Printer size={18} />
            </button>
            <button onClick={() => shareAppContent(settings.businessProfile.name, 'نظام صفوان لإدارة المخازن')} className="p-2 text-slate-400 hover:text-slate-900 transition-all" title="مشاركة">
              <Share2 size={18} />
            </button>
            <div className="w-px h-4 bg-slate-200 mx-2"></div>
            <div className="w-8 h-8 rounded-full bg-amber-400 flex items-center justify-center text-slate-900 font-black text-xs border border-white shadow-sm">
              S1
            </div>
          </div>
        </header>

        <div className="flex-1 overflow-y-auto p-4 lg:p-6 no-scrollbar bg-[#fcfdfe]">
          {activeTab === Tab.DASHBOARD && (
            <Dashboard 
              accounts={accounts} 
              products={products} 
              stockIn={stockInOrders} 
              stockOut={stockOutOrders} 
              settings={settings} 
              onNavigate={setActiveTab} 
              onNavigateWithFilter={handleNavigateWithFilter} 
            />
          )}
          {activeTab === Tab.ACCOUNTS && <AccountsScreen accounts={accounts} setAccounts={setAccounts} settings={settings} />}
          {activeTab === Tab.GROUPS && <GroupsScreen groups={productGroups} setGroups={setProductGroups} products={products} settings={settings} />}
          {activeTab === Tab.PRODUCTS && <ProductsScreen products={products} setProducts={setProducts} groups={productGroups} />}
          {activeTab === Tab.PRODUCT_LIST && (
            <ProductListScreen 
              products={products} 
              setProducts={setProducts} 
              settings={settings} 
              initialFilter={activeProductFilter} 
              groups={productGroups} 
              onAddClick={() => setActiveTab(Tab.PRODUCTS)} 
            />
          )}
          {activeTab === Tab.STOCK_IN && <StockInScreen accounts={accounts} products={products} orders={stockInOrders} setOrders={setStockInOrders} setProducts={setProducts} />}
          {activeTab === Tab.STOCK_OUT && <StockOutScreen accounts={accounts} products={products} orders={stockOutOrders} setOrders={setStockOutOrders} setProducts={setProducts} />}
          {activeTab === Tab.INVENTORY && <InventoryScreen products={products} records={inventoryRecords} setRecords={setInventoryRecords} setProducts={setProducts} groups={productGroups} />}
          {activeTab === Tab.REPORTS && <ReportsScreen products={products} stockIn={stockInOrders} stockOut={stockOutOrders} accounts={accounts} settings={settings} />}
          {activeTab === Tab.SETTINGS && (
            <SettingsScreen 
              settings={settings} 
              setSettings={setSettings} 
              allData={{ accounts, groups: productGroups, products, stockIn: stockInOrders, stockOut: stockOutOrders, inventory: inventoryRecords }} 
              onImportData={d => { 
                setAccounts(d.accounts || []); 
                setProductGroups(d.groups || []); 
                setProducts(d.products || []); 
                setStockInOrders(d.stockIn || []);
                setStockOutOrders(d.stockOut || []);
                setInventoryRecords(d.inventory || []);
                if (d.settings) setSettings(d.settings); 
              }} 
            />
          )}
        </div>
      </main>
      
      {/* غطاء معتم عند فتح القائمة في الموبايل */}
      {isSidebarOpen && (
        <div className="fixed inset-0 bg-slate-950/50 backdrop-blur-sm z-40 lg:hidden" onClick={() => setIsSidebarOpen(false)}></div>
      )}
    </div>
  );
};

export default App;
