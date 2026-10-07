<!DOCTYPE html>
<html lang="zh-TW">
<head>
   <meta charset="UTF-8">
   <meta name="viewport" content="width=device-width, initial-scale=1.0">
   <title>台股策略追蹤小工具</title>
   <!-- Tailwind CSS CDN -->
   <script src="https://cdn.tailwindcss.com"></script>
   <style>
       body { background-color: #f3f4f6; }
   </style>
</head>
<body class="p-3 max-w-lg mx-auto text-gray-800">

   <!-- 標題 -->
   <header class="mb-4 text-center">
       <h1 class="text-2xl font-bold text-blue-600">📈 策略持股追蹤</h1>
       <p class="text-xs text-gray-500 mt-1">族群連動 2.0 & VCP 2.0 專用管理工具</p>
   </header>

   <!-- 新增股票按鈕 -->
   <button onclick="openModal()" class="w-full bg-blue-600 hover:bg-blue-700 text-white font-bold py-3 px-4 rounded-xl shadow-md text-lg active:scale-95 transition mb-4">
       ＋ 新增持股記錄
   </button>

   <!-- 股票列表容器 -->
   <div id="stockList" class="space-y-3 pb-12">
       <!-- 動態帶入 -->
   </div>

   <!-- 新增 / 編輯 表單 Modal -->
   <div id="stockModal" class="fixed inset-0 bg-black bg-opacity-50 hidden flex items-center justify-center p-4 z-50">
       <div class="bg-white rounded-2xl w-full max-w-md p-5 shadow-2xl">
           <h2 id="modalTitle" class="text-xl font-bold mb-4 text-gray-700">新增持股</h2>
           <form id="stockForm" onsubmit="saveStock(event)">
               <input type="hidden" id="stockId">
               
               <div class="mb-3">
                   <label class="block text-sm font-semibold mb-1">股票名稱 / 代號</label>
                   <input type="text" id="name" required placeholder="例如: 2330 台積電" class="w-full border rounded-xl p-3 text-lg focus:ring-2 focus:ring-blue-500 outline-none">
               </div>

               <div class="mb-3">
                   <label class="block text-sm font-semibold mb-1">策略選擇</label>
                   <select id="strategy" class="w-full border rounded-xl p-3 text-lg focus:ring-2 focus:ring-blue-500 outline-none bg-white">
                       <option value="族群連動 2.0">族群連動 2.0 (持有10天 / 停利35%)</option>
                       <option value="VCP 2.0">VCP 2.0 (持有14天 / 停利25%)</option>
                   </select>
               </div>

               <div class="mb-3">
                   <label class="block text-sm font-semibold mb-1">買進日期 (第0天)</label>
                   <input type="date" id="buyDate" required class="w-full border rounded-xl p-3 text-lg focus:ring-2 focus:ring-blue-500 outline-none">
               </div>

               <div class="mb-3">
                   <label class="block text-sm font-semibold mb-1">買進成本 (元)</label>
                   <input type="number" step="0.01" id="cost" required placeholder="例如: 100" class="w-full border rounded-xl p-3 text-lg focus:ring-2 focus:ring-blue-500 outline-none">
               </div>

               <div class="mb-3">
                   <label class="block text-sm font-semibold mb-1">目前股價 (元)</label>
                   <input type="number" step="0.01" id="currentPrice" required placeholder="例如: 110" class="w-full border rounded-xl p-3 text-lg focus:ring-2 focus:ring-blue-500 outline-none">
               </div>

               <div class="flex space-x-3 mt-5">
                   <button type="button" onclick="closeModal()" class="w-1/2 bg-gray-200 hover:bg-gray-300 py-3 rounded-xl font-bold text-gray-700 active:scale-95 transition">取消</button>
                   <button type="submit" class="w-1/2 bg-blue-600 hover:bg-blue-700 text-white py-3 rounded-xl font-bold active:scale-95 transition">儲存</button>
               </div>
           </form>
       </div>
   </div>

   <!-- 邏輯與互動 Script -->
   <script>
       let stocks = JSON.parse(localStorage.getItem('stocks_data')) || [];

       // 初始化載入
       window.onload = function() {
           renderStocks();
       };

       function openModal(id = null) {
           document.getElementById('stockModal').classList.remove('hidden');
           if (id) {
               document.getElementById('modalTitle').innerText = '編輯持股';
               const stock = stocks.find(s => s.id === id);
               document.getElementById('stockId').value = stock.id;
               document.getElementById('name').value = stock.name;
               document.getElementById('strategy').value = stock.strategy;
               document.getElementById('buyDate').value = stock.buyDate;
               document.getElementById('cost').value = stock.cost;
               document.getElementById('currentPrice').value = stock.currentPrice;
           } else {
               document.getElementById('modalTitle').innerText = '新增持股';
               document.getElementById('stockForm').reset();
               document.getElementById('stockId').value = '';
               // 預設帶入今天日期
               document.getElementById('buyDate').valueAsDate = new Date();
           }
       }

       function closeModal() {
           document.getElementById('stockModal').classList.add('hidden');
       }

       function saveStock(e) {
           e.preventDefault();
           const id = document.getElementById('stockId').value;
           const name = document.getElementById('name').value;
           const strategy = document.getElementById('strategy').value;
           const buyDate = document.getElementById('buyDate').value;
           const cost = parseFloat(document.getElementById('cost').value);
           const currentPrice = parseFloat(document.getElementById('currentPrice').value);

           if (id) {
               // 編輯
               const index = stocks.findIndex(s => s.id == id);
               if (index !== -1) {
                   stocks[index] = { ...stocks[index], name, strategy, buyDate, cost, currentPrice };
               }
           } else {
               // 新增
               const newStock = {
                   id: Date.now(),
                   name,
                   strategy,
                   buyDate,
                   cost,
                   currentPrice
               };
               stocks.push(newStock);
           }

           saveAndRender();
           closeModal();
       }

       function deleteStock(id) {
           if (confirm('確定要刪除這筆紀錄嗎？')) {
               stocks = stocks.filter(s => s.id !== id);
               saveAndRender();
           }
       }

       function saveAndRender() {
           localStorage.setItem('stocks_data', JSON.stringify(stocks));
           renderStocks();
       }

       // 計算交易日天數與日期工具 (簡化以自然日/交易日對應，這裡以持有天數邏輯計算)
       function calculateStockMetrics(stock) {
           const buy = new Date(stock.buyDate);
           const today = new Date();
           // 計算經過的自然日天數作為簡易持有天數 (亦可依需求調整為計算日曆天)
           const diffTime = today - buy;
           const diffDays = Math.floor(diffTime / (1000 * 60 * 60 * 24));
           const heldDays = diffDays >= 0 ? diffDays : 0;

           const maxDays = stock.strategy === '族群連動 2.0' ? 10 : 14;
           const targetProfitRate = stock.strategy === '族群連動 2.0' ? 0.35 : 0.25;

           // 目標價
           const targetPrice = stock.cost * (1 + targetProfitRate);

           // 到期日計算 (以買進日 + maxDays 天數概算)
           let expiryDate = new Date(buy);
           expiryDate.setDate(expiryDate.getDate() + maxDays);
           const expiryDateStr = expiryDate.toISOString().split('T')[0];

           // 目前報酬率
           const returnRate = (stock.currentPrice - stock.cost) / stock.cost;

           // 判斷狀態：是否達停利標準 或 快到期 (剩餘 <= 1天 或 已超過)
           const isTargetReached = stock.currentPrice >= targetPrice;
           const isExpiringSoon = heldDays >= (maxDays - 1);
           const isPriority = isTargetReached || isExpiringSoon;

           return {
               heldDays,
               maxDays,
               targetPrice,
               expiryDateStr,
               returnRate,
               isTargetReached,
               isExpiringSoon,
               isPriority
           };
       }

       function renderStocks() {
           const container = document.getElementById('stockList');
           if (stocks.length === 0) {
               container.innerHTML = `<div class="text-center py-10 text-gray-400 bg-white rounded-2xl shadow-sm">目前沒有持股記錄，請點擊上方按鈕新增</div>`;
               return;
           }

           // 排序：符合優先條件 (達停利或快到期) 的排在最前面
           const processedStocks = stocks.map(stock => ({
               ...stock,
               metrics: calculateStockMetrics(stock)
           }));

           processedStocks.sort((a, b) => {
               if (a.metrics.isPriority && !b.metrics.isPriority) return -1;
               if (!a.metrics.isPriority && b.metrics.isPriority) return 1;
               return b.id - -a.id; // 較新的排前面
           });

           let html = '';
           processedStocks.forEach(s => {
               const m = s.metrics;
               const returnPercent = (m.returnRate * 100).toFixed(2);
               const returnColor = m.returnRate >= 0 ? 'text-red-500' : 'text-green-500'; // 台股紅漲綠跌
               
               // 警示標籤
               let badge = '';
               let cardBorder = 'border-gray-200';
               if (m.isTargetReached) {
                   badge = `<span class="bg-red-100 text-red-600 text-xs font-bold px-2 py-1 rounded-full animate-pulse">🔥 已達停利標準 (+${(m.targetPrice/s.cost*100-100).toFixed(0)}%)</span>`;
                   cardBorder = 'border-red-400 bg-red-50/30';
               } else if (m.isExpiringSoon) {
                   badge = `<span class="bg-amber-100 text-amber-700 text-xs font-bold px-2 py-1 rounded-full">⚠️ 即將到期 (${m.heldDays}/${m.maxDays}天)</span>`;
                   cardBorder = 'border-amber-400 bg-amber-50/30';
               }

               html += `
                   <div class="bg-white rounded-2xl p-4 shadow-md border-2 ${cardBorder} transition">
                       <div class="flex justify-between items-start mb-2">
