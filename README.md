# saladamix
<!DOCTYPE html>
<html lang="pt-PT">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>Saladamix - Pedidos Online</title>
  
  <!-- Tailwind CSS para o Design -->
  <script src="https://cdn.tailwindcss.com"></script>
  
  <!-- React e ReactDOM -->
  <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
  <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
  
  <!-- Babel para converter o código no navegador -->
  <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

  <style>
    /* Previne seleção de texto acidental ao clicar nos botões */
    button { user-select: none; -webkit-tap-highlight-color: transparent; }
    body { overscroll-behavior-y: none; }
  </style>
</head>
<body class="bg-slate-100">

  <div id="root"></div>

  <script type="text/babel">
    const { useState, useMemo } = React;

    // Ícones em formato SVG (para não depender de bibliotecas externas)
    const IconCheck = () => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="3" strokeLinecap="round" strokeLinejoin="round" width="1em" height="1em"><polyline points="20 6 9 17 4 12"></polyline></svg>;
    const IconX = () => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" width="1em" height="1em"><line x1="18" y1="6" x2="6" y2="18"></line><line x1="6" y1="6" x2="18" y2="18"></line></svg>;
    const IconSmartphone = () => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" width="1em" height="1em"><rect x="5" y="2" width="14" height="20" rx="2" ry="2"></rect><line x1="12" y1="18" x2="12.01" y2="18"></line></svg>;
    const IconShoppingBag = () => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" width="1em" height="1em"><path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"></path><line x1="3" y1="6" x2="21" y2="6"></line><path d="M16 10a4 4 0 0 1-8 0"></path></svg>;
    const IconMessageCircle = () => <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" strokeWidth="2" strokeLinecap="round" strokeLinejoin="round" width="1em" height="1em"><path d="M21 11.5a8.38 8.38 0 0 1-.9 3.8 8.5 8.5 0 0 1-7.6 4.7 8.38 8.38 0 0 1-3.8-.9L3 21l1.9-5.7a8.38 8.38 0 0 1-.9-3.8 8.5 8.5 0 0 1 4.7-7.6 8.38 8.38 0 0 1 3.8-.9h.5a8.48 8.48 0 0 1 8 8v.5z"></path></svg>;

    // Configuração do WhatsApp
    const WHATSAPP_NUMBER = "553196885832"; 

    const SIZES = [
      { id: '300', name: '300ML', desc: '3 frutas ou mais', price: 7.00, specialLimit: 0 },
      { id: '400', name: '400ML', desc: '3 frutas + 1 especial', price: 10.00, specialLimit: 1 },
      { id: '500', name: '500ML', desc: '3 frutas + 2 especiais', price: 12.00, specialLimit: 2 },
      { id: 'trad', name: 'TRADICIONAL', desc: 'Pote 300ML Pronto', price: 5.00, specialLimit: 0 },
    ];

    const FRUITS = ['Banana', 'Laranja', 'Maçã', 'Goiaba', 'Mamão', 'Abacaxi', 'Manga'];
    const SPECIAL_FRUITS = ['Uva', 'Morango', 'Kiwi'];
    const TOPPINGS = [
      'Leite em Pó', 'Creme de Leite', 'Calda Leite Ninho', 'Leite Condensado',
      'Sorvete de Morango', 'Amendoim', 'Granola', 'Aveia'
    ];

    function App() {
      const [selectedSize, setSelectedSize] = useState(null);
      const [selectedFruits, setSelectedFruits] = useState([]);
      const [selectedSpecialFruits, setSelectedSpecialFruits] = useState([]);
      const [selectedToppings, setSelectedToppings] = useState([]);
      const [showSummary, setShowSummary] = useState(false);
      const [customerName, setCustomerName] = useState("");

      const { total } = useMemo(() => {
        if (!selectedSize) return { total: 0 };
        let currentTotal = selectedSize.price;
        const extraCount = Math.max(0, selectedToppings.length - 3);
        currentTotal += (extraCount * 1.00);
        return { total: currentTotal };
      }, [selectedSize, selectedToppings]);

      const isTradicional = selectedSize?.id === 'trad';

      const handleSizeChange = (size) => {
        setSelectedSize(size);
        if (selectedSpecialFruits.length > size.specialLimit) {
          setSelectedSpecialFruits(selectedSpecialFruits.slice(0, size.specialLimit));
        }
      };

      const handleFruitToggle = (fruit) => {
        setSelectedFruits(prev => prev.includes(fruit) ? prev.filter(f => f !== fruit) : [...prev, fruit]);
      };

      const handleSpecialFruitToggle = (fruit) => {
        if (selectedSpecialFruits.includes(fruit)) {
          setSelectedSpecialFruits(prev => prev.filter(f => f !== fruit));
        } else {
          if (selectedSize && selectedSpecialFruits.length < selectedSize.specialLimit) {
            setSelectedSpecialFruits([...selectedSpecialFruits, fruit]);
          }
        }
      };

      const handleToppingToggle = (topping) => {
        setSelectedToppings(prev => prev.includes(topping) ? prev.filter(t => t !== topping) : [...prev, topping]);
      };

      const checkValidationAndShowSummary = () => {
        if (!selectedSize) return alert("Escolha o tamanho da salada primeiro!");
        setShowSummary(true);
      };

      const sendToWhatsApp = () => {
        if (!customerName.trim()) {
          alert("Por favor, informe o seu nome.");
          return;
        }

        let message = `*NOVO PEDIDO - SALADAMIX* 🍓\n`;
        message += `---------------------------\n`;
        message += `*Cliente:* ${customerName}\n\n`;
        message += `*Tamanho:* ${selectedSize.name} (R$ ${selectedSize.price.toFixed(2).replace('.', ',')})\n`;
        
        if (!isTradicional) {
          const allFruits = [...selectedFruits, ...selectedSpecialFruits.map(f => `${f} (Especial)`)];
          message += `*Frutas:* ${allFruits.length > 0 ? allFruits.join(', ') : 'Nenhuma'}\n`;
        } else {
          message += `*Frutas:* Salada Tradicional\n`;
        }

        if (selectedToppings.length > 0) {
          message += `*Acompanhamentos:* \n`;
          selectedToppings.forEach((t, index) => {
            message += `  - ${t} ${index >= 3 ? '(+ R$ 1,00)' : '(Grátis)'}\n`;
          });
        } else {
           message += `*Acompanhamentos:* Nenhum\n`;
        }

        message += `---------------------------\n`;
        message += `*TOTAL A PAGAR: R$ ${total.toFixed(2).replace('.', ',')}*\n`;

        const encodedMessage = encodeURIComponent(message);
        const whatsappUrl = `https://wa.me/${WHATSAPP_NUMBER}?text=${encodedMessage}`;
        window.open(whatsappUrl, '_blank');
      };

      return (
        <div className="min-h-screen bg-slate-100 font-sans text-slate-800 pb-28 md:max-w-md md:mx-auto md:shadow-2xl md:min-h-[100dvh]">
          <header className="bg-white px-4 py-4 shadow-sm sticky top-0 z-10 flex items-center justify-between border-b-[3px] border-pink-500">
            <div className="flex items-center gap-2">
              <span className="text-3xl">🍓</span>
              <div>
                <h1 className="text-2xl font-black text-slate-900 tracking-tight leading-none">Salada<span className="text-pink-600">mix</span></h1>
                <p className="text-green-600 font-bold text-[10px] tracking-widest uppercase mt-0.5">App de Pedidos</p>
              </div>
            </div>
            <div className="text-slate-300 text-2xl"><IconSmartphone /></div>
          </header>

          <main className="p-4 space-y-4">
            <section className="bg-white rounded-2xl p-4 shadow-sm border border-slate-200">
              <h2 className="text-sm font-bold text-pink-600 mb-3 flex items-center gap-2">
                <span className="bg-pink-100 text-pink-600 w-5 h-5 rounded-full flex items-center justify-center text-[10px]">1</span> TAMANHO
              </h2>
              <div className="space-y-2">
                {SIZES.map(size => (
                  <button key={size.id} onClick={() => handleSizeChange(size)} className={`w-full flex items-center justify-between p-3 rounded-xl border-2 text-left ${selectedSize?.id === size.id ? 'border-pink-500 bg-pink-50' : 'border-slate-100 bg-white'}`}>
                    <div>
                      <div className={`font-bold ${selectedSize?.id === size.id ? 'text-pink-700' : 'text-slate-700'}`}>{size.name}</div>
                      <div className="text-[10px] text-slate-500">{size.desc}</div>
                    </div>
                    <div className="font-black text-slate-800">R$ {size.price.toFixed(2).replace('.', ',')}</div>
                  </button>
                ))}
              </div>
            </section>

            <section className={`bg-white rounded-2xl p-4 shadow-sm border border-slate-200 ${(!selectedSize || isTradicional) ? 'opacity-50 pointer-events-none' : ''}`}>
              <h2 className="text-sm font-bold text-green-600 mb-3 flex items-center gap-2">
                <span className="bg-green-100 text-green-600 w-5 h-5 rounded-full flex items-center justify-center text-[10px]">2</span> FRUTAS
              </h2>
              <div className="grid grid-cols-2 gap-2 mb-3">
                {FRUITS.map(fruit => (
                  <button key={fruit} onClick={() => handleFruitToggle(fruit)} className={`p-2 rounded-lg border text-xs font-medium flex justify-between items-center ${selectedFruits.includes(fruit) ? 'border-green-500 bg-green-50 text-green-700' : 'border-slate-200 bg-white text-slate-600'}`}>
                    {fruit} {selectedFruits.includes(fruit) && <div className="text-green-500 text-lg"><IconCheck /></div>}
                  </button>
                ))}
              </div>
              <div className="bg-slate-50 rounded-xl p-3 border border-slate-100">
                <h3 className="text-[11px] font-bold text-pink-600 mb-2 flex justify-between items-center">
                  <span>★ ESPECIAIS</span>
                  <span className="bg-pink-100 px-2 py-0.5 rounded-full text-[9px]">{selectedSpecialFruits.length} / {selectedSize?.specialLimit || 0}</span>
                </h3>
                <div className="grid grid-cols-2 gap-2">
                  {SPECIAL_FRUITS.map(fruit => {
                    const isSelected = selectedSpecialFruits.includes(fruit);
                    const isMaxed = !isSelected && selectedSize && selectedSpecialFruits.length >= selectedSize.specialLimit;
                    return (
                      <button key={fruit} onClick={() => handleSpecialFruitToggle(fruit)} disabled={isMaxed} className={`p-2 rounded-lg border text-xs flex justify-between items-center ${isSelected ? 'border-pink-500 bg-pink-100 text-pink-700 font-bold' : isMaxed ? 'border-slate-100 bg-slate-100 text-slate-400 opacity-50' : 'border-slate-200 bg-white text-slate-600'}`}>
                        {fruit} {isSelected && <div className="text-pink-500 text-lg"><IconCheck /></div>}
                      </button>
                    );
                  })}
                </div>
              </div>
            </section>

            <section className={`bg-white rounded-2xl p-4 shadow-sm border border-slate-200 ${!selectedSize ? 'opacity-50 pointer-events-none' : ''}`}>
               <div className="flex justify-between items-center mb-3">
                 <h2 className="text-sm font-bold text-blue-600 flex items-center gap-2">
                   <span className="bg-blue-100 text-blue-600 w-5 h-5 rounded-full flex items-center justify-center text-[10px]">3</span> EXTRAS
                 </h2>
                 <span className="text-[9px] bg-blue-50 text-blue-600 px-2 py-1 rounded font-bold">Até 3 Grátis</span>
              </div>
              <div className="grid grid-cols-1 gap-2">
                {TOPPINGS.map((topping) => {
                  const isSelected = selectedToppings.includes(topping);
                  const isExtra = isSelected && selectedToppings.indexOf(topping) >= 3;
                  return (
                    <button key={topping} onClick={() => handleToppingToggle(topping)} className={`p-3 rounded-xl border flex items-center justify-between ${isSelected ? 'border-blue-500 bg-blue-50' : 'border-slate-200 bg-white'}`}>
                      <span className={`text-xs ${isSelected ? 'font-bold text-blue-800' : 'text-slate-600'}`}>{topping}</span>
                      <div className="flex items-center gap-2">
                        {isExtra && <span className="text-[10px] font-bold text-pink-500">+R$ 1,00</span>}
                        <div className={`w-4 h-4 rounded border flex items-center justify-center ${isSelected ? 'border-blue-500 bg-blue-500' : 'border-slate-300'}`}>
                          {isSelected && <div className="text-white text-xs"><IconCheck /></div>}
                        </div>
                      </div>
                    </button>
                  );
                })}
              </div>
            </section>
          </main>

          <div className="fixed bottom-0 left-0 w-full bg-white border-t border-slate-200 p-4 z-40 md:max-w-md md:left-1/2 md:-translate-x-1/2 rounded-t-2xl">
            <div className="flex items-center justify-between">
              <div>
                <p className="text-[10px] text-slate-500 font-bold uppercase mb-0.5">Total</p>
                <div className="text-xl font-black text-slate-900">R$ {total.toFixed(2).replace('.', ',')}</div>
              </div>
              <button onClick={checkValidationAndShowSummary} className={`px-6 py-3 rounded-full font-bold shadow-lg flex items-center gap-2 text-sm ${selectedSize ? 'bg-green-500 hover:bg-green-600 text-white shadow-green-200' : 'bg-slate-200 text-slate-400'}`}>
                <span className="text-lg"><IconShoppingBag /></span> Avançar
              </button>
            </div>
          </div>

          {showSummary && (
            <div className="fixed inset-0 bg-slate-900/60 z-50 flex items-end justify-center sm:items-center">
              <div className="bg-white w-full h-[85vh] sm:h-auto sm:max-w-md rounded-t-3xl sm:rounded-3xl flex flex-col shadow-2xl overflow-hidden">
                <div className="p-4 border-b flex justify-between items-center bg-slate-50">
                  <h2 className="text-base font-black text-slate-900">Finalizar Pedido</h2>
                  <button onClick={() => setShowSummary(false)} className="bg-slate-200 p-2 rounded-full text-lg"><IconX /></button>
                </div>
                <div className="p-5 overflow-y-auto flex-1 bg-white space-y-4">
                  <div>
                    <label className="block text-xs font-bold text-slate-500 uppercase mb-2">Seu Nome</label>
                    <input type="text" value={customerName} onChange={(e) => setCustomerName(e.target.value)} placeholder="Digite o seu nome..." className="w-full bg-slate-100 border-none p-4 rounded-xl font-medium focus:ring-2 focus:ring-pink-500 outline-none"/>
                  </div>
                </div>
                <div className="p-5 bg-white border-t border-slate-100 pb-8">
                  <div className="flex justify-between items-center mb-4 px-2">
                    <span className="font-bold text-slate-500 uppercase text-xs">Total a Pagar</span>
                    <span className="font-black text-2xl text-slate-900">R$ {total.toFixed(2).replace('.', ',')}</span>
                  </div>
                  <button onClick={sendToWhatsApp} className="w-full bg-[#25D366] hover:bg-[#128C7E] text-white py-4 rounded-2xl font-black shadow-xl flex items-center justify-center gap-2">
                    <span className="text-xl"><IconMessageCircle /></span> Enviar para o WhatsApp
                  </button>
                </div>
              </div>
            </div>
          )}
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById('root'));
    root.render(<App />);
  </script>
</body>
</html>
