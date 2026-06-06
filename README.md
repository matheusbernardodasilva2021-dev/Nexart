# Nexart
import { useState, useEffect } from "react";

const INITIAL_PRODUCTS = [
  { id: 1, name: "Air Max Futura", brand: "Nike", price: 899.90, originalPrice: 1199.90, category: "Tênis", rating: 4.8, reviews: 2341, img: "👟", badge: "OFERTA", color: "#FF6B35", stock: 15, sold: 87 },
  { id: 2, name: "Galaxy S25 Ultra", brand: "Samsung", price: 6499.00, originalPrice: 7299.00, category: "Eletrônicos", rating: 4.9, reviews: 5123, img: "📱", badge: "NOVO", color: "#6C63FF", stock: 8, sold: 203 },
  { id: 3, name: "MacBook Air M3", brand: "Apple", price: 10999.00, originalPrice: 12499.00, category: "Eletrônicos", rating: 4.9, reviews: 3892, img: "💻", badge: "TOP", color: "#00C9A7", stock: 5, sold: 145 },
  { id: 4, name: "Camiseta Premium Fit", brand: "Hering", price: 89.90, originalPrice: 149.90, category: "Roupas", rating: 4.3, reviews: 871, img: "👕", badge: "40% OFF", color: "#FF6B6B", stock: 42, sold: 312 },
  { id: 5, name: "Perfume Noir Intense", brand: "Natura", price: 299.00, originalPrice: 399.00, category: "Beleza", rating: 4.7, reviews: 1204, img: "🌸", badge: "OFERTA", color: "#F7B731", stock: 20, sold: 98 },
  { id: 6, name: "Smart TV 55\" QLED", brand: "LG", price: 4299.00, originalPrice: 5499.00, category: "Eletrônicos", rating: 4.6, reviews: 2087, img: "📺", badge: "22% OFF", color: "#45AAF2", stock: 3, sold: 67 },
];

const CATEGORIES = ["Todos", "Eletrônicos", "Tênis", "Roupas", "Beleza", "Casa", "Acessórios"];
const EMOJIS = ["👟","📱","💻","👕","🌸","📺","🛋️","🎧","👜","⌚","🪑","🏃","🎮","📷","🧴","👒","🛍️","💄","🎒","⌨️"];
const COLORS = ["#FF6B35","#6C63FF","#00C9A7","#FF6B6B","#F7B731","#45AAF2","#A55EEA","#26de81","#FC5C65","#0fb9b1","#8854d0","#FD9644"];
const BADGES = ["NOVO","OFERTA","TOP","DESTAQUE","EXCLUSIVO","LIMITADO"];

const banners = [
  { title: "Black Friday Antecipado", subtitle: "Até 60% OFF em tudo", bg: ["#1a1a2e","#16213e"], accent: "#FF6B35", emoji: "🔥" },
  { title: "Eletrônicos em Promoção", subtitle: "As melhores marcas", bg: ["#0f3460","#533483"], accent: "#00C9A7", emoji: "⚡" },
  { title: "Moda & Estilo", subtitle: "Nova coleção com frete grátis", bg: ["#1a0533","#2d1b69"], accent: "#F7B731", emoji: "✨" },
];

function Stars({ rating }) {
  return (
    <span style={{ display: "inline-flex", gap: 1 }}>
      {[1,2,3,4,5].map(i => <span key={i} style={{ fontSize: 11, color: i <= Math.round(rating) ? "#F7B731" : "#ddd" }}>★</span>)}
    </span>
  );
}

function fmt(p) { return p.toLocaleString("pt-BR", { style: "currency", currency: "BRL" }); }
function disc(p) { return Math.round((1 - p.price / p.originalPrice) * 100); }

// ── SELLER PANEL ──────────────────────────────────────────────
function SellerPanel({ products, setProducts, onBack }) {
  const [view, setView] = useState("dashboard"); // dashboard | list | form
  const [editProduct, setEditProduct] = useState(null);
  const [form, setForm] = useState({ name:"", brand:"", price:"", originalPrice:"", category:"Eletrônicos", img:"📱", badge:"NOVO", color:"#FF6B35", stock:"", sold:"0" });
  const [notification, setNotification] = useState(null);

  const notify = (msg, emoji="✅") => { setNotification({msg,emoji}); setTimeout(()=>setNotification(null),2500); };

  const totalRevenue = products.reduce((s,p)=>s+p.price*p.sold,0);
  const totalSold = products.reduce((s,p)=>s+p.sold,0);
  const lowStock = products.filter(p=>p.stock<=5).length;

  const openNew = () => { setForm({ name:"",brand:"",price:"",originalPrice:"",category:"Eletrônicos",img:"📱",badge:"NOVO",color:"#FF6B35",stock:"",sold:"0" }); setEditProduct(null); setView("form"); };
  const openEdit = (p) => { setForm({...p, price:String(p.price), originalPrice:String(p.originalPrice), stock:String(p.stock), sold:String(p.sold)}); setEditProduct(p); setView("form"); };

  const saveProduct = () => {
    if (!form.name || !form.price || !form.originalPrice || !form.stock) { notify("Preencha todos os campos!", "⚠️"); return; }
    const data = { ...form, price: parseFloat(form.price), originalPrice: parseFloat(form.originalPrice), stock: parseInt(form.stock), sold: parseInt(form.sold)||0, rating: editProduct?.rating||4.5, reviews: editProduct?.reviews||0 };
    if (editProduct) {
      setProducts(prev => prev.map(p => p.id === editProduct.id ? {...data, id:editProduct.id} : p));
      notify("Produto atualizado!", "✏️");
    } else {
      setProducts(prev => [...prev, {...data, id: Date.now()}]);
      notify("Produto cadastrado!", "🎉");
    }
    setView("list");
  };

  const deleteProduct = (id) => { setProducts(prev=>prev.filter(p=>p.id!==id)); notify("Produto removido","🗑️"); };

  const inp = (field, value) => setForm(f => ({...f, [field]:value}));

  const inputStyle = { width:"100%", padding:"11px 14px", border:"1.5px solid #eee", borderRadius:12, fontSize:13, outline:"none", boxSizing:"border-box", background:"#fafafa" };
  const labelStyle = { fontSize:12, fontWeight:700, color:"#555", marginBottom:4, display:"block" };

  return (
    <div style={{ background:"#f5f5f7", minHeight:"100vh", maxWidth:430, margin:"0 auto", fontFamily:"'Segoe UI',system-ui,sans-serif" }}>

      {notification && (
        <div style={{ position:"fixed",top:20,left:"50%",transform:"translateX(-50%)",background:"#111",color:"#fff",padding:"12px 20px",borderRadius:30,zIndex:9999,fontSize:13,fontWeight:600,display:"flex",alignItems:"center",gap:8,boxShadow:"0 8px 32px rgba(0,0,0,0.3)",whiteSpace:"nowrap",animation:"slideDown .3s ease" }}>
          {notification.emoji} {notification.msg}
        </div>
      )}

      {/* Header */}
      <div style={{ background:"#111",padding:"16px 18px 14px",display:"flex",alignItems:"center",gap:12 }}>
        <button onClick={onBack} style={{ background:"rgba(255,255,255,0.1)",border:"none",color:"#fff",borderRadius:10,padding:"6px 12px",fontSize:12,cursor:"pointer",fontWeight:600 }}>← Loja</button>
        <div style={{ flex:1 }}>
          <div style={{ color:"#fff",fontWeight:900,fontSize:16 }}>⚙️ Painel do Vendedor</div>
          <div style={{ color:"rgba(255,255,255,0.5)",fontSize:11 }}>Gerencie seus produtos</div>
        </div>
        {view !== "form" && (
          <button onClick={openNew} style={{ background:"#FF6B35",border:"none",color:"#fff",borderRadius:12,padding:"8px 14px",fontSize:12,fontWeight:700,cursor:"pointer" }}>+ Novo</button>
        )}
      </div>

      {/* Sub Nav */}
      {view !== "form" && (
        <div style={{ background:"#fff",display:"flex",borderBottom:"1px solid #f0f0f0" }}>
          {[["dashboard","📊 Dashboard"],["list","📦 Produtos"]].map(([k,l])=>(
            <button key={k} onClick={()=>setView(k)} style={{ flex:1,padding:"12px 0",border:"none",background:"none",fontSize:13,fontWeight:700,cursor:"pointer",color:view===k?"#FF6B35":"#aaa",borderBottom:view===k?"2px solid #FF6B35":"2px solid transparent" }}>{l}</button>
          ))}
        </div>
      )}

      {/* DASHBOARD */}
      {view === "dashboard" && (
        <div style={{ padding:16 }}>
          <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:12,marginBottom:16 }}>
            {[
              { label:"Receita Total", value:fmt(totalRevenue), icon:"💰", color:"#00C9A7", bg:"#00C9A715" },
              { label:"Produtos Vendidos", value:totalSold, icon:"📦", color:"#6C63FF", bg:"#6C63FF15" },
              { label:"Total de Produtos", value:products.length, icon:"🏷️", color:"#FF6B35", bg:"#FF6B3515" },
              { label:"Estoque Baixo", value:lowStock, icon:"⚠️", color:"#FC5C65", bg:"#FC5C6515" },
            ].map(card=>(
              <div key={card.label} style={{ background:"#fff",borderRadius:16,padding:16,boxShadow:"0 2px 8px rgba(0,0,0,0.06)" }}>
                <div style={{ fontSize:28,marginBottom:6 }}>{card.icon}</div>
                <div style={{ fontSize:18,fontWeight:900,color:card.color }}>{card.value}</div>
                <div style={{ fontSize:11,color:"#888",marginTop:2 }}>{card.label}</div>
              </div>
            ))}
          </div>

          <div style={{ background:"#fff",borderRadius:16,padding:16,boxShadow:"0 2px 8px rgba(0,0,0,0.06)",marginBottom:12 }}>
            <div style={{ fontSize:14,fontWeight:800,marginBottom:12 }}>🏆 Top Produtos</div>
            {[...products].sort((a,b)=>b.sold-a.sold).slice(0,4).map(p=>(
              <div key={p.id} style={{ display:"flex",alignItems:"center",gap:10,marginBottom:10 }}>
                <div style={{ fontSize:28,background:`${p.color}15`,borderRadius:10,width:44,height:44,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0 }}>{p.img}</div>
                <div style={{ flex:1,minWidth:0 }}>
                  <div style={{ fontSize:13,fontWeight:700,color:"#111",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap" }}>{p.name}</div>
                  <div style={{ fontSize:11,color:"#888" }}>{p.sold} vendas · {fmt(p.price)}</div>
                  <div style={{ background:"#f0f0f0",borderRadius:4,height:4,marginTop:4,overflow:"hidden" }}>
                    <div style={{ background:p.color,height:"100%",width:`${Math.min(100,(p.sold/300)*100)}%`,borderRadius:4 }} />
                  </div>
                </div>
              </div>
            ))}
          </div>

          {lowStock > 0 && (
            <div style={{ background:"#FFF3CD",borderRadius:16,padding:16,border:"1px solid #FFE083" }}>
              <div style={{ fontSize:13,fontWeight:800,color:"#856404",marginBottom:8 }}>⚠️ Estoque Baixo</div>
              {products.filter(p=>p.stock<=5).map(p=>(
                <div key={p.id} style={{ display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:6 }}>
                  <span style={{ fontSize:12 }}>{p.img} {p.name}</span>
                  <span style={{ fontSize:12,fontWeight:700,color:"#FC5C65",background:"#FC5C6515",padding:"2px 8px",borderRadius:6 }}>{p.stock} un.</span>
                </div>
              ))}
            </div>
          )}
        </div>
      )}

      {/* PRODUCT LIST */}
      {view === "list" && (
        <div style={{ padding:16 }}>
          {products.length === 0 && (
            <div style={{ textAlign:"center",padding:"60px 0",color:"#aaa" }}>
              <div style={{ fontSize:60 }}>📦</div>
              <div style={{ fontWeight:700,marginTop:12 }}>Nenhum produto ainda</div>
              <button onClick={openNew} style={{ marginTop:12,background:"#FF6B35",border:"none",color:"#fff",borderRadius:12,padding:"10px 24px",fontSize:13,fontWeight:700,cursor:"pointer" }}>+ Cadastrar Produto</button>
            </div>
          )}
          {products.map(p=>(
            <div key={p.id} style={{ background:"#fff",borderRadius:16,padding:14,marginBottom:12,boxShadow:"0 2px 8px rgba(0,0,0,0.06)",display:"flex",gap:12,alignItems:"center" }}>
              <div style={{ fontSize:36,background:`${p.color}20`,borderRadius:12,width:52,height:52,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0 }}>{p.img}</div>
              <div style={{ flex:1,minWidth:0 }}>
                <div style={{ fontSize:13,fontWeight:800,color:"#111",overflow:"hidden",textOverflow:"ellipsis",whiteSpace:"nowrap" }}>{p.name}</div>
                <div style={{ fontSize:12,color:p.color,fontWeight:600 }}>{fmt(p.price)}</div>
                <div style={{ display:"flex",gap:8,marginTop:4 }}>
                  <span style={{ fontSize:10,background:"#f0f0f0",padding:"2px 7px",borderRadius:6,color:"#666" }}>📦 {p.stock} un.</span>
                  <span style={{ fontSize:10,background:"#00C9A715",padding:"2px 7px",borderRadius:6,color:"#00C9A7" }}>✓ {p.sold} vendas</span>
                </div>
              </div>
              <div style={{ display:"flex",flexDirection:"column",gap:6 }}>
                <button onClick={()=>openEdit(p)} style={{ background:"#6C63FF15",border:"none",color:"#6C63FF",borderRadius:8,padding:"6px 12px",fontSize:11,fontWeight:700,cursor:"pointer" }}>✏️ Editar</button>
                <button onClick={()=>deleteProduct(p.id)} style={{ background:"#FC5C6515",border:"none",color:"#FC5C65",borderRadius:8,padding:"6px 12px",fontSize:11,fontWeight:700,cursor:"pointer" }}>🗑️ Excluir</button>
              </div>
            </div>
          ))}
        </div>
      )}

      {/* FORM */}
      {view === "form" && (
        <div style={{ padding:16 }}>
          <div style={{ display:"flex",alignItems:"center",gap:10,marginBottom:16 }}>
            <button onClick={()=>setView("list")} style={{ background:"#eee",border:"none",borderRadius:10,padding:"6px 12px",fontSize:12,cursor:"pointer",fontWeight:600 }}>← Voltar</button>
            <div style={{ fontWeight:900,fontSize:16 }}>{editProduct?"Editar Produto":"Novo Produto"}</div>
          </div>

          {/* Preview */}
          <div style={{ background:`linear-gradient(135deg,${form.color}15,${form.color}30)`,borderRadius:20,padding:"20px 0",textAlign:"center",fontSize:64,marginBottom:16 }}>{form.img}</div>

          <div style={{ background:"#fff",borderRadius:16,padding:16,boxShadow:"0 2px 8px rgba(0,0,0,0.06)",display:"flex",flexDirection:"column",gap:14 }}>

            <div><label style={labelStyle}>Nome do Produto *</label>
              <input style={inputStyle} value={form.name} onChange={e=>inp("name",e.target.value)} placeholder="Ex: Tênis Air Max 2025" /></div>

            <div><label style={labelStyle}>Marca *</label>
              <input style={inputStyle} value={form.brand} onChange={e=>inp("brand",e.target.value)} placeholder="Ex: Nike, Adidas..." /></div>

            <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:10 }}>
              <div><label style={labelStyle}>Preço (R$) *</label>
                <input style={inputStyle} type="number" value={form.price} onChange={e=>inp("price",e.target.value)} placeholder="0,00" /></div>
              <div><label style={labelStyle}>Preço Original *</label>
                <input style={inputStyle} type="number" value={form.originalPrice} onChange={e=>inp("originalPrice",e.target.value)} placeholder="0,00" /></div>
            </div>

            <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:10 }}>
              <div><label style={labelStyle}>Estoque *</label>
                <input style={inputStyle} type="number" value={form.stock} onChange={e=>inp("stock",e.target.value)} placeholder="0" /></div>
              <div><label style={labelStyle}>Categoria</label>
                <select style={inputStyle} value={form.category} onChange={e=>inp("category",e.target.value)}>
                  {CATEGORIES.filter(c=>c!=="Todos").map(c=><option key={c}>{c}</option>)}
                </select></div>
            </div>

            <div><label style={labelStyle}>Badge</label>
              <div style={{ display:"flex",gap:6,flexWrap:"wrap" }}>
                {BADGES.map(b=>(
                  <button key={b} onClick={()=>inp("badge",b)} style={{ padding:"5px 12px",borderRadius:8,border:"1.5px solid",borderColor:form.badge===b?"#FF6B35":"#eee",background:form.badge===b?"#FF6B3510":"#fff",color:form.badge===b?"#FF6B35":"#666",fontSize:11,fontWeight:700,cursor:"pointer" }}>{b}</button>
                ))}
              </div>
            </div>

            <div><label style={labelStyle}>Emoji do Produto</label>
              <div style={{ display:"flex",gap:6,flexWrap:"wrap" }}>
                {EMOJIS.map(e=>(
                  <button key={e} onClick={()=>inp("img",e)} style={{ fontSize:22,background:form.img===e?"#FF6B3515":"#f5f5f7",border:form.img===e?"2px solid #FF6B35":"2px solid transparent",borderRadius:10,width:40,height:40,cursor:"pointer" }}>{e}</button>
                ))}
              </div>
            </div>

            <div><label style={labelStyle}>Cor do Card</label>
              <div style={{ display:"flex",gap:8,flexWrap:"wrap" }}>
                {COLORS.map(c=>(
                  <button key={c} onClick={()=>inp("color",c)} style={{ width:30,height:30,borderRadius:"50%",background:c,border:form.color===c?"3px solid #111":"3px solid transparent",cursor:"pointer" }} />
                ))}
              </div>
            </div>

            <button onClick={saveProduct} style={{ width:"100%",padding:"15px 0",borderRadius:14,border:"none",background:"linear-gradient(135deg,#FF6B35,#ff4500)",color:"#fff",fontSize:15,fontWeight:800,cursor:"pointer",boxShadow:"0 6px 20px rgba(255,107,53,0.3)",marginTop:4 }}>
              {editProduct ? "✓ Salvar Alterações" : "🎉 Cadastrar Produto"}
            </button>
          </div>
          <div style={{ height:40 }} />
        </div>
      )}

      <style>{`@keyframes slideDown{from{opacity:0;transform:translate(-50%,-20px)}to{opacity:1;transform:translate(-50%,0)}}::-webkit-scrollbar{display:none}`}</style>
    </div>
  );
}

// ── MAIN STORE ────────────────────────────────────────────────
export default function ShoppingApp() {
  const [products, setProducts] = useState(INITIAL_PRODUCTS);
  const [cart, setCart] = useState([]);
  const [wishlist, setWishlist] = useState([]);
  const [search, setSearch] = useState("");
  const [category, setCategory] = useState("Todos");
  const [cartOpen, setCartOpen] = useState(false);
  const [bannerIdx, setBannerIdx] = useState(0);
  const [notification, setNotification] = useState(null);
  const [activeTab, setActiveTab] = useState("home");
  const [selectedProduct, setSelectedProduct] = useState(null);
  const [sortBy, setSortBy] = useState("default");
  const [sellerMode, setSellerMode] = useState(false);

  useEffect(() => {
    const t = setInterval(() => setBannerIdx(i => (i + 1) % banners.length), 4000);
    return () => clearInterval(t);
  }, []);

  const notify = (msg, emoji="✅") => { setNotification({msg,emoji}); setTimeout(()=>setNotification(null),2500); };

  const addToCart = (product) => {
    setCart(prev => {
      const ex = prev.find(i=>i.id===product.id);
      if (ex) return prev.map(i=>i.id===product.id?{...i,qty:i.qty+1}:i);
      return [...prev, {...product,qty:1}];
    });
    notify(`${product.name} adicionado!`,"🛒");
  };

  const toggleWishlist = (product) => {
    if (!product?.id) return;
    setWishlist(prev=>{
      if (prev.find(i=>i.id===product.id)) { notify("Removido dos favoritos","💔"); return prev.filter(i=>i.id!==product.id); }
      notify("Adicionado aos favoritos!","❤️"); return [...prev,product];
    });
  };

  const removeFromCart = (id) => setCart(prev=>prev.filter(i=>i.id!==id));
  const updateQty = (id,delta) => setCart(prev=>prev.map(i=>i.id===id?{...i,qty:Math.max(1,i.qty+delta)}:i));
  const cartTotal = cart.reduce((s,i)=>s+i.price*i.qty,0);
  const cartCount = cart.reduce((s,i)=>s+i.qty,0);

  let filtered = products.filter(p=>
    (category==="Todos"||p.category===category) &&
    (p.name.toLowerCase().includes(search.toLowerCase())||p.brand.toLowerCase().includes(search.toLowerCase()))
  );
  if (sortBy==="priceAsc") filtered=[...filtered].sort((a,b)=>a.price-b.price);
  if (sortBy==="priceDesc") filtered=[...filtered].sort((a,b)=>b.price-a.price);
  if (sortBy==="rating") filtered=[...filtered].sort((a,b)=>b.rating-a.rating);

  if (sellerMode) return <SellerPanel products={products} setProducts={setProducts} onBack={()=>setSellerMode(false)} />;

  const banner = banners[bannerIdx];

  return (
    <div style={{ fontFamily:"'Segoe UI',system-ui,sans-serif",background:"#f5f5f7",minHeight:"100vh",maxWidth:430,margin:"0 auto",position:"relative",overflowX:"hidden" }}>

      {notification && (
        <div style={{ position:"fixed",top:20,left:"50%",transform:"translateX(-50%)",background:"#111",color:"#fff",padding:"12px 20px",borderRadius:30,zIndex:9999,fontSize:13,fontWeight:600,display:"flex",alignItems:"center",gap:8,boxShadow:"0 8px 32px rgba(0,0,0,0.3)",animation:"slideDown .3s ease",whiteSpace:"nowrap" }}>
          <span>{notification.emoji}</span> {notification.msg}
        </div>
      )}

      {/* HEADER */}
      <div style={{ background:"#fff",padding:"14px 18px 10px",position:"sticky",top:0,zIndex:100,boxShadow:"0 1px 12px rgba(0,0,0,0.08)" }}>
        <div style={{ display:"flex",justifyContent:"space-between",alignItems:"center",marginBottom:12 }}>
          <div>
            <div style={{ fontSize:20,fontWeight:900,letterSpacing:-1,color:"#111" }}>✦ <span style={{ color:"#FF6B35" }}>Nex</span>art</div>
            <div style={{ fontSize:11,color:"#888",marginTop:-2 }}>A nova era do shopping digital</div>
          </div>
          <div style={{ display:"flex",gap:8,alignItems:"center" }}>
            <button onClick={()=>setSellerMode(true)} style={{ background:"#111",border:"none",color:"#fff",borderRadius:10,padding:"7px 12px",fontSize:11,fontWeight:700,cursor:"pointer" }}>⚙️ Vendedor</button>
            <button onClick={()=>setCartOpen(true)} style={{ background:"#FF6B35",border:"none",color:"#fff",borderRadius:20,padding:"8px 14px",fontSize:13,fontWeight:700,cursor:"pointer",display:"flex",alignItems:"center",gap:6 }}>
              🛒 {cartCount>0&&<span style={{ background:"#fff",color:"#FF6B35",borderRadius:"50%",width:18,height:18,fontSize:10,fontWeight:900,display:"flex",alignItems:"center",justifyContent:"center" }}>{cartCount}</span>}
            </button>
          </div>
        </div>
        <div style={{ position:"relative" }}>
          <span style={{ position:"absolute",left:12,top:"50%",transform:"translateY(-50%)",fontSize:16 }}>🔍</span>
          <input value={search} onChange={e=>setSearch(e.target.value)} placeholder="Buscar produtos, marcas..."
            style={{ width:"100%",padding:"10px 12px 10px 38px",border:"1.5px solid #eee",borderRadius:14,fontSize:13,outline:"none",background:"#f9f9f9",boxSizing:"border-box" }}
            onFocus={e=>e.target.style.border="1.5px solid #FF6B35"} onBlur={e=>e.target.style.border="1.5px solid #eee"} />
        </div>
      </div>

      {/* BANNER */}
      <div style={{ margin:"16px 16px 0",borderRadius:20,overflow:"hidden",position:"relative",height:160 }}>
        {banners.map((b,i)=>(
          <div key={i} style={{ position:"absolute",inset:0,transition:"opacity .6s ease",opacity:i===bannerIdx?1:0,background:`linear-gradient(135deg,${b.bg[0]},${b.bg[1]})`,padding:24,display:"flex",flexDirection:"column",justifyContent:"flex-end" }}>
            <div style={{ fontSize:32 }}>{b.emoji}</div>
            <div style={{ color:b.accent,fontWeight:900,fontSize:18,lineHeight:1.2 }}>{b.title}</div>
            <div style={{ color:"rgba(255,255,255,0.8)",fontSize:12,marginTop:4 }}>{b.subtitle}</div>
            <button style={{ marginTop:10,background:b.accent,color:"#fff",border:"none",borderRadius:20,padding:"7px 18px",fontSize:11,fontWeight:700,cursor:"pointer",alignSelf:"flex-start" }}>Ver ofertas →</button>
          </div>
        ))}
        <div style={{ position:"absolute",bottom:12,right:14,display:"flex",gap:5 }}>
          {banners.map((_,i)=>(
            <div key={i} onClick={()=>setBannerIdx(i)} style={{ width:i===bannerIdx?20:6,height:6,borderRadius:3,background:i===bannerIdx?"#fff":"rgba(255,255,255,0.4)",cursor:"pointer",transition:"all .3s" }} />
          ))}
        </div>
      </div>

      {/* CATEGORIES */}
      <div style={{ padding:"14px 16px 0" }}>
        <div style={{ display:"flex",gap:8,overflowX:"auto",paddingBottom:4 }}>
          {CATEGORIES.map(cat=>(
            <button key={cat} onClick={()=>setCategory(cat)} style={{ padding:"7px 16px",borderRadius:20,border:"none",cursor:"pointer",background:category===cat?"#FF6B35":"#fff",color:category===cat?"#fff":"#555",fontSize:12,fontWeight:700,whiteSpace:"nowrap",boxShadow:category===cat?"0 4px 12px rgba(255,107,53,0.3)":"0 1px 4px rgba(0,0,0,0.08)",transition:"all .2s" }}>{cat}</button>
          ))}
        </div>
      </div>

      {/* SORT */}
      <div style={{ padding:"12px 16px 0",display:"flex",justifyContent:"space-between",alignItems:"center" }}>
        <div style={{ fontSize:13,color:"#888" }}><strong style={{ color:"#111" }}>{filtered.length}</strong> produtos</div>
        <select value={sortBy} onChange={e=>setSortBy(e.target.value)} style={{ fontSize:12,border:"1.5px solid #eee",borderRadius:10,padding:"5px 10px",background:"#fff",color:"#555",outline:"none" }}>
          <option value="default">Relevância</option>
          <option value="priceAsc">Menor preço</option>
          <option value="priceDesc">Maior preço</option>
          <option value="rating">Melhor avaliado</option>
        </select>
      </div>

      {/* PRODUCTS */}
      <div style={{ padding:16,display:"grid",gridTemplateColumns:"1fr 1fr",gap:12 }}>
        {filtered.length===0&&(
          <div style={{ gridColumn:"1/-1",textAlign:"center",padding:"40px 0",color:"#aaa" }}>
            <div style={{ fontSize:48 }}>🔍</div>
            <div style={{ fontWeight:700,marginTop:8 }}>Nenhum produto encontrado</div>
          </div>
        )}
        {filtered.map(product=>{
          const inWishlist=wishlist.find(i=>i.id===product.id);
          const inCart=cart.find(i=>i.id===product.id);
          return (
            <div key={product.id} onClick={()=>setSelectedProduct(product)} style={{ background:"#fff",borderRadius:18,overflow:"hidden",boxShadow:"0 2px 12px rgba(0,0,0,0.07)",cursor:"pointer",transition:"transform .2s,box-shadow .2s" }}
              onMouseEnter={e=>{e.currentTarget.style.transform="translateY(-2px)";e.currentTarget.style.boxShadow="0 8px 24px rgba(0,0,0,0.12)"}}
              onMouseLeave={e=>{e.currentTarget.style.transform="";e.currentTarget.style.boxShadow="0 2px 12px rgba(0,0,0,0.07)"}}>
              <div style={{ background:`linear-gradient(135deg,${product.color}15,${product.color}30)`,padding:"20px 0 16px",textAlign:"center",position:"relative" }}>
                <div style={{ fontSize:52 }}>{product.img}</div>
                <div style={{ position:"absolute",top:8,left:8,background:product.color,color:"#fff",fontSize:9,fontWeight:800,padding:"3px 7px",borderRadius:6 }}>{product.badge}</div>
                <button onClick={e=>{e.stopPropagation();toggleWishlist(product)}} style={{ position:"absolute",top:8,right:8,background:inWishlist?"#FF6B6B":"rgba(255,255,255,0.9)",border:"none",borderRadius:"50%",width:28,height:28,fontSize:14,display:"flex",alignItems:"center",justifyContent:"center",cursor:"pointer",boxShadow:"0 2px 6px rgba(0,0,0,0.1)" }}>
                  {inWishlist?"❤️":"🤍"}
                </button>
                {product.stock<=5&&<div style={{ position:"absolute",bottom:4,right:6,fontSize:9,color:"#FC5C65",fontWeight:700 }}>⚠️ {product.stock} un.</div>}
              </div>
              <div style={{ padding:"10px 12px 12px" }}>
                <div style={{ fontSize:10,color:product.color,fontWeight:700,marginBottom:2 }}>{product.brand}</div>
                <div style={{ fontSize:13,fontWeight:700,color:"#111",lineHeight:1.3,marginBottom:6,overflow:"hidden",display:"-webkit-box",WebkitLineClamp:2,WebkitBoxOrient:"vertical" }}>{product.name}</div>
                <div style={{ display:"flex",alignItems:"center",gap:4,marginBottom:4 }}>
                  <Stars rating={product.rating} />
                  <span style={{ fontSize:10,color:"#888" }}>({product.reviews?.toLocaleString()})</span>
                </div>
                <div style={{ fontSize:10,color:"#aaa",textDecoration:"line-through" }}>{fmt(product.originalPrice)}</div>
                <div style={{ fontSize:16,fontWeight:900,color:"#111",marginBottom:2 }}>{fmt(product.price)}</div>
                <div style={{ fontSize:10,color:"#2ecc71",fontWeight:700,marginBottom:8 }}>-{disc(product)}% OFF</div>
                <button onClick={e=>{e.stopPropagation();addToCart(product)}} style={{ width:"100%",padding:"9px 0",borderRadius:12,border:"none",background:inCart?"#2ecc71":"#FF6B35",color:"#fff",fontSize:12,fontWeight:700,cursor:"pointer",transition:"background .2s" }}>
                  {inCart?"✓ No carrinho":"Adicionar"}
                </button>
              </div>
            </div>
          );
        })}
      </div>

      {/* BOTTOM NAV */}
      <div style={{ background:"#fff",position:"fixed",bottom:0,left:"50%",transform:"translateX(-50%)",width:"100%",maxWidth:430,display:"flex",justifyContent:"space-around",padding:"10px 0 16px",borderTop:"1px solid #f0f0f0",zIndex:200 }}>
        {[["🏠","Início","home"],["🔍","Buscar","search"],["❤️","Favoritos","wishlist"],["👤","Conta","profile"]].map(([icon,label,key])=>(
          <button key={key} onClick={()=>setActiveTab(key)} style={{ background:"none",border:"none",cursor:"pointer",display:"flex",flexDirection:"column",alignItems:"center",gap:2 }}>
            <span style={{ fontSize:22 }}>{icon}</span>
            <span style={{ fontSize:10,color:activeTab===key?"#FF6B35":"#aaa",fontWeight:activeTab===key?700:400 }}>{label}</span>
          </button>
        ))}
      </div>

      <div style={{ height:80 }} />

      {/* CART DRAWER */}
      {cartOpen&&(
        <div style={{ position:"fixed",inset:0,zIndex:500 }}>
          <div onClick={()=>setCartOpen(false)} style={{ position:"absolute",inset:0,background:"rgba(0,0,0,0.5)" }} />
          <div style={{ position:"absolute",right:0,top:0,bottom:0,width:"90%",maxWidth:400,background:"#fff",display:"flex",flexDirection:"column" }}>
            <div style={{ padding:"20px 20px 14px",borderBottom:"1px solid #f0f0f0",display:"flex",justifyContent:"space-between",alignItems:"center" }}>
              <div style={{ fontSize:18,fontWeight:900 }}>🛒 Meu Carrinho</div>
              <button onClick={()=>setCartOpen(false)} style={{ background:"#f5f5f7",border:"none",borderRadius:"50%",width:32,height:32,fontSize:18,cursor:"pointer" }}>✕</button>
            </div>
            <div style={{ flex:1,overflowY:"auto",padding:16 }}>
              {cart.length===0?(
                <div style={{ textAlign:"center",padding:"60px 0",color:"#aaa" }}>
                  <div style={{ fontSize:64 }}>🛒</div>
                  <div style={{ fontWeight:700,marginTop:12 }}>Carrinho vazio</div>
                </div>
              ):cart.map(item=>(
                <div key={item.id} style={{ display:"flex",gap:12,marginBottom:16,background:"#f9f9f9",borderRadius:14,padding:12,alignItems:"center" }}>
                  <div style={{ fontSize:40,background:`${item.color}20`,borderRadius:12,width:56,height:56,display:"flex",alignItems:"center",justifyContent:"center",flexShrink:0 }}>{item.img}</div>
                  <div style={{ flex:1,minWidth:0 }}>
                    <div style={{ fontSize:13,fontWeight:700,color:"#111" }}>{item.name}</div>
                    <div style={{ fontSize:12,color:item.color,fontWeight:600 }}>{fmt(item.price)}</div>
                    <div style={{ display:"flex",alignItems:"center",gap:8,marginTop:6 }}>
                      <button onClick={()=>updateQty(item.id,-1)} style={{ background:"#eee",border:"none",borderRadius:6,width:24,height:24,fontSize:14,cursor:"pointer",fontWeight:700 }}>-</button>
                      <span style={{ fontWeight:700,fontSize:13 }}>{item.qty}</span>
                      <button onClick={()=>updateQty(item.id,1)} style={{ background:"#FF6B35",border:"none",borderRadius:6,width:24,height:24,fontSize:14,cursor:"pointer",fontWeight:700,color:"#fff" }}>+</button>
                    </div>
                  </div>
                  <button onClick={()=>removeFromCart(item.id)} style={{ background:"none",border:"none",fontSize:18,cursor:"pointer",color:"#ccc" }}>🗑️</button>
                </div>
              ))}
            </div>
            {cart.length>0&&(
              <div style={{ padding:20,borderTop:"1px solid #f0f0f0" }}>
                <div style={{ display:"flex",justifyContent:"space-between",marginBottom:6 }}><span style={{ color:"#888",fontSize:13 }}>Subtotal</span><span style={{ fontWeight:700 }}>{fmt(cartTotal)}</span></div>
                <div style={{ display:"flex",justifyContent:"space-between",marginBottom:6 }}><span style={{ color:"#888",fontSize:13 }}>Frete</span><span style={{ fontWeight:700,color:"#2ecc71" }}>GRÁTIS</span></div>
                <div style={{ display:"flex",justifyContent:"space-between",marginBottom:16,paddingTop:10,borderTop:"1px dashed #eee" }}><span style={{ fontWeight:900,fontSize:16 }}>Total</span><span style={{ fontWeight:900,fontSize:18,color:"#FF6B35" }}>{fmt(cartTotal)}</span></div>
                <button onClick={()=>{notify("Pedido realizado! 🎉","🎉");setCart([]);setCartOpen(false)}} style={{ width:"100%",padding:"15px 0",borderRadius:16,border:"none",background:"linear-gradient(135deg,#FF6B35,#ff4500)",color:"#fff",fontSize:15,fontWeight:800,cursor:"pointer",boxShadow:"0 6px 20px rgba(255,107,53,0.4)" }}>
                  Finalizar Compra →
                </button>
                <div style={{ textAlign:"center",marginTop:10,fontSize:11,color:"#aaa" }}>🔒 Pagamento 100% seguro</div>
              </div>
            )}
          </div>
        </div>
      )}

      {/* PRODUCT MODAL */}
      {selectedProduct&&(
        <div style={{ position:"fixed",inset:0,zIndex:500 }}>
          <div onClick={()=>setSelectedProduct(null)} style={{ position:"absolute",inset:0,background:"rgba(0,0,0,0.6)" }} />
          <div style={{ position:"absolute",bottom:0,left:"50%",transform:"translateX(-50%)",width:"100%",maxWidth:430,background:"#fff",borderRadius:"24px 24px 0 0",padding:"24px 20px 40px" }}>
            <div style={{ textAlign:"center" }}>
              <div style={{ width:40,height:4,background:"#eee",borderRadius:2,margin:"0 auto 20px" }} />
              <div style={{ background:`linear-gradient(135deg,${selectedProduct.color}15,${selectedProduct.color}30)`,borderRadius:20,padding:"24px 0",fontSize:80,marginBottom:16 }}>{selectedProduct.img}</div>
              <div style={{ fontSize:11,color:selectedProduct.color,fontWeight:700 }}>{selectedProduct.brand}</div>
              <div style={{ fontSize:20,fontWeight:900,margin:"6px 0 8px",color:"#111" }}>{selectedProduct.name}</div>
              <div style={{ display:"flex",justifyContent:"center",alignItems:"center",gap:6,marginBottom:12 }}>
                <Stars rating={selectedProduct.rating} />
                <span style={{ fontSize:12,color:"#888" }}>{selectedProduct.rating} ({selectedProduct.reviews?.toLocaleString()} avaliações)</span>
              </div>
              <div style={{ fontSize:13,color:"#aaa",textDecoration:"line-through" }}>{fmt(selectedProduct.originalPrice)}</div>
              <div style={{ fontSize:28,fontWeight:900,color:"#111" }}>{fmt(selectedProduct.price)}</div>
              <div style={{ fontSize:13,color:"#2ecc71",fontWeight:700,marginBottom:20 }}>Você economiza {fmt(selectedProduct.originalPrice-selectedProduct.price)}</div>
              <div style={{ display:"grid",gridTemplateColumns:"1fr 1fr",gap:10 }}>
                <button onClick={()=>toggleWishlist(selectedProduct)} style={{ padding:"14px 0",borderRadius:14,border:"1.5px solid #eee",background:"#fff",fontSize:14,fontWeight:700,cursor:"pointer",color:"#555" }}>
                  {wishlist.find(i=>i.id===selectedProduct.id)?"❤️ Favorito":"🤍 Favoritar"}
                </button>
                <button onClick={()=>{addToCart(selectedProduct);setSelectedProduct(null)}} style={{ padding:"14px 0",borderRadius:14,border:"none",background:"linear-gradient(135deg,#FF6B35,#ff4500)",color:"#fff",fontSize:14,fontWeight:700,cursor:"pointer" }}>🛒 Adicionar</button>
              </div>
            </div>
          </div>
        </div>
      )}

      <style>{`@keyframes slideDown{from{opacity:0;transform:translate(-50%,-20px)}to{opacity:1;transform:translate(-50%,0)}}::-webkit-scrollbar{display:none}`}</style>
    </div>
  );
}
