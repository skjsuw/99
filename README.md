<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes">
    <title>Nova 量化引擎 · 实时运行日志</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        html, body {
            width: 100%;
            height: 100%;
            overflow: hidden;
        }

        body {
            background: radial-gradient(circle at 30% 20%, #0f0c29, #1a1a3e, #24243e);
            font-family: 'Inter', 'Segoe UI', system-ui, -apple-system, 'Poppins', sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 12px;
        }

        .profit-card {
            width: 100%;
            height: 100%;
            max-width: 1600px;
            max-height: 1000px;
            background: rgba(16, 14, 38, 0.92);
            backdrop-filter: blur(18px);
            border-radius: 40px;
            padding: 16px 24px 22px;
            box-shadow: 0 30px 60px rgba(0, 0, 0, 0.7), 0 0 0 1px rgba(120, 80, 255, 0.25);
            border: 1px solid rgba(140, 100, 255, 0.3);
            display: flex;
            flex-direction: column;
        }

        /* ===== 第一行：余额/盈利设置 ===== */
        .row-settings {
            display: flex;
            align-items: center;
            gap: 14px;
            flex-shrink: 0;
            padding-bottom: 10px;
            border-bottom: 1px solid rgba(140, 100, 255, 0.1);
            flex-wrap: wrap;
        }

        .settings-group {
            display: flex;
            align-items: center;
            gap: 12px;
            flex-wrap: wrap;
        }

        .setting-item {
            display: flex;
            align-items: center;
            gap: 6px;
            background: rgba(0, 0, 0, 0.35);
            padding: 4px 14px 4px 12px;
            border-radius: 30px;
            border: 1px solid rgba(140, 100, 255, 0.15);
        }

        .setting-item .icon {
            font-size: 14px;
        }

        .setting-item label {
            font-size: 11px;
            color: #7a6e9e;
            font-weight: 500;
        }

        .setting-item input {
            background: transparent;
            border: none;
            color: #d4c8ff;
            font-family: 'JetBrains Mono', monospace;
            font-size: 15px;
            font-weight: 700;
            width: 80px;
            outline: none;
            padding: 2px 0;
        }

        .setting-item input:focus {
            border-bottom: 2px solid #8a6cff;
        }

        .setting-item .unit {
            font-size: 10px;
            color: #5f5290;
            font-weight: 500;
        }

        .setting-btn {
            background: rgba(120, 80, 255, 0.15);
            border: 1px solid rgba(140, 100, 255, 0.3);
            padding: 5px 20px;
            border-radius: 30px;
            font-size: 12px;
            font-weight: 600;
            cursor: pointer;
            color: #c8b8ff;
            transition: 0.2s;
        }

        .setting-btn:hover {
            background: rgba(120, 80, 255, 0.3);
        }

        .time-box {
            background: rgba(0, 0, 0, 0.35);
            border-radius: 40px;
            padding: 4px 18px;
            border: 1px solid rgba(140, 100, 255, 0.2);
            backdrop-filter: blur(4px);
            flex-shrink: 0;
            margin-left: auto;
        }

        .time-box .date {
            color: #b8a5ff;
            font-size: 13px;
            font-family: 'JetBrains Mono', monospace;
            font-weight: 500;
            letter-spacing: 0.3px;
        }

        /* ===== 第二行：标题 + 状态 ===== */
        .row-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-shrink: 0;
            padding: 8px 0 8px 0;
            flex-wrap: wrap;
            gap: 8px;
        }

        .title-section {
            display: flex;
            align-items: center;
            gap: 14px;
            flex-wrap: wrap;
        }

        .title {
            font-size: 22px;
            font-weight: 700;
            background: linear-gradient(135deg, #f0eaff 0%, #b7a4ff 50%, #7a5cff 100%);
            -webkit-background-clip: text;
            background-clip: text;
            color: transparent;
            letter-spacing: -0.2px;
        }

        .badge {
            font-size: 11px;
            color: #8a7eb5;
            background: rgba(100, 60, 200, 0.15);
            padding: 2px 14px;
            border-radius: 20px;
            border: 1px solid rgba(140, 100, 255, 0.2);
        }

        .status-right {
            display: flex;
            gap: 10px;
            align-items: center;
            flex-wrap: wrap;
        }

        .status-item {
            display: flex;
            align-items: center;
            gap: 6px;
            font-size: 12px;
            color: #8a7eb5;
        }

        .status-item .dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            display: inline-block;
        }

        .dot.green { background: #6effb0; box-shadow: 0 0 8px rgba(110, 255, 176, 0.3); }
        .dot.yellow { background: #ffd87a; box-shadow: 0 0 8px rgba(255, 216, 122, 0.2); }
        .dot.purple { background: #b8a5ff; box-shadow: 0 0 8px rgba(184, 165, 255, 0.2); }

        .status-value {
            color: #d4c8ff;
            font-weight: 600;
            font-family: monospace;
            font-size: 13px;
        }

        .action-btn {
            background: rgba(100, 60, 200, 0.1);
            border: 1px solid rgba(140, 100, 255, 0.25);
            padding: 4px 14px;
            border-radius: 30px;
            font-size: 11px;
            font-weight: 500;
            cursor: pointer;
            color: #b8a5ff;
            transition: 0.2s;
        }

        .action-btn:hover {
            background: rgba(100, 60, 200, 0.25);
        }

        .action-btn.danger {
            border-color: rgba(255, 90, 90, 0.25);
            color: #ff8a8a;
        }

        .action-btn.danger:hover {
            background: rgba(255, 70, 70, 0.15);
        }

        .action-btn.profit {
            border-color: rgba(140, 100, 255, 0.4);
            color: #c8b8ff;
        }

        .action-btn.profit:hover {
            background: rgba(120, 80, 255, 0.2);
        }

        /* ===== 日志区域 ===== */
        .log-area {
            background: rgba(6, 4, 20, 0.7);
            border-radius: 24px;
            padding: 16px 20px;
            flex: 1;
            min-height: 0;
            overflow-y: auto;
            font-family: 'JetBrains Mono', 'SF Mono', monospace;
            font-size: 13px;
            border: 1px solid #2a2850;
            box-shadow: inset 0 0 10px rgba(0,0,0,0.5);
            margin-top: 4px;
        }

        .log-line {
            padding: 5px 0;
            border-bottom: 1px solid rgba(31, 29, 66, 0.5);
            font-size: 13px;
            line-height: 1.5;
            display: flex;
            flex-wrap: wrap;
            gap: 4px 10px;
        }

        .log-time {
            color: #5f5290;
            font-weight: 500;
            flex-shrink: 0;
            min-width: 70px;
        }

        .log-scan { color: #8ab8ff; }
        .log-gas { color: #ffbc6e; }
        .log-profit { color: #8affb0; font-weight: 600; }
        .log-arbitrage { color: #ffd87a; background: rgba(255,210,90,0.06); padding: 1px 6px; border-radius: 12px;}
        .log-warning { color: #ffaa66; }
        .log-execute { color: #6effd4; }
        .log-info { color: #b8a5ff; }
        .log-success { color: #6effb0; }

        .log-area::-webkit-scrollbar {
            width: 5px;
        }
        .log-area::-webkit-scrollbar-track {
            background: #0e0c24;
            border-radius: 10px;
        }
        .log-area::-webkit-scrollbar-thumb {
            background: #3d3570;
            border-radius: 10px;
        }

        /* Modal */
        .modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.85);
            backdrop-filter: blur(10px);
            z-index: 1000;
            justify-content: center;
            align-items: center;
        }

        .modal-content {
            background: #14122e;
            border-radius: 40px;
            max-width: 500px;
            width: 90%;
            max-height: 70vh;
            border: 1px solid rgba(140, 100, 255, 0.35);
            box-shadow: 0 20px 50px rgba(0,0,0,0.6);
            display: flex;
            flex-direction: column;
            animation: fadeInUp 0.2s ease;
        }

        @keyframes fadeInUp {
            from { opacity: 0; transform: translateY(20px);}
            to { opacity: 1; transform: translateY(0);}
        }

        .modal-header {
            padding: 18px 24px;
            border-bottom: 1px solid #2a2850;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .modal-header h3 {
            color: #b8a5ff;
            font-size: 18px;
        }

        .modal-close {
            background: rgba(255,70,70,0.15);
            border: none;
            color: #ff8a8a;
            font-size: 22px;
            cursor: pointer;
            width: 30px;
            height: 30px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            transition: 0.2s;
        }

        .modal-close:hover {
            background: rgba(255,70,70,0.4);
        }

        .profit-list {
            padding: 16px 20px;
            overflow-y: auto;
            flex: 1;
        }

        .profit-record {
            background: rgba(10, 8, 28, 0.6);
            border-radius: 16px;
            padding: 12px 14px;
            margin-bottom: 10px;
            border-left: 3px solid #8a6cff;
        }

        .profit-record .record-time {
            color: #8a7eb5;
            font-size: 11px;
            font-family: monospace;
            margin-bottom: 4px;
        }

        .profit-record .record-detail {
            color: #d4c8ff;
            font-size: 13px;
            display: flex;
            justify-content: space-between;
            flex-wrap: wrap;
            gap: 6px;
        }

        .record-amount {
            color: #8affb0;
            font-weight: bold;
            font-size: 15px;
        }

        .empty-profit {
            text-align: center;
            color: #5f5290;
            padding: 40px;
        }

        .modal-footer {
            padding: 14px 24px;
            border-top: 1px solid #2a2850;
            font-size: 12px;
            color: #6f6299;
            text-align: center;
        }

        .profit-list::-webkit-scrollbar {
            width: 5px;
        }
        .profit-list::-webkit-scrollbar-track {
            background: #0e0c24;
            border-radius: 10px;
        }
        .profit-list::-webkit-scrollbar-thumb {
            background: #3d3570;
            border-radius: 10px;
        }

        input[type="number"]::-webkit-inner-spin-button,
        input[type="number"]::-webkit-outer-spin-button {
            opacity: 0.4;
        }

        @media (max-width: 768px) {
            .profit-card { padding: 12px 14px 16px; border-radius: 24px; }
            .title { font-size: 17px; }
            .log-area { padding: 10px 12px; font-size: 11px; }
            .log-line { font-size: 11px; padding: 4px 0; }
            .row-settings { flex-direction: column; align-items: stretch; gap: 8px; }
            .settings-group { flex-wrap: wrap; }
            .time-box { margin-left: 0; align-self: flex-start; }
            .row-header { flex-direction: column; align-items: stretch; gap: 6px; }
            .status-right { flex-wrap: wrap; }
            .badge { font-size: 10px; }
            .log-time { min-width: 55px; }
            .setting-item input { width: 60px; font-size: 13px; }
        }
    </style>
</head>
<body>
<div class="profit-card">
    <!-- 第一行：余额/盈利设置 -->
    <div class="row-settings">
        <div class="settings-group">
            <div class="setting-item">
                <span class="icon">💰</span>
                <label>余额</label>
                <input type="number" id="balanceInput" step="0.01" value="1988.00">
                <span class="unit">USDT</span>
            </div>
            <div class="setting-item">
                <span class="icon">📈</span>
                <label>盈利</label>
                <input type="number" id="profitInput" step="0.01" value="1877.94">
                <span class="unit">USDT</span>
            </div>
            <button class="setting-btn" id="updateBtn">更新</button>
        </div>
        <div class="time-box">
            <span class="date" id="runningTime">--</span>
        </div>
    </div>

    <!-- 第二行：标题 + 状态 -->
    <div class="row-header">
        <div class="title-section">
            <span class="title">✦ Nova 量化引擎</span>
            <span class="badge">实时运行日志</span>
        </div>
        <div class="status-right">
            <div class="status-item">
                <span class="dot green"></span>
                <span>状态</span>
                <span class="status-value" id="statusText">运行中</span>
            </div>
            <div class="status-item">
                <span class="dot purple"></span>
                <span>余额</span>
                <span class="status-value" id="displayBalance">1,988.00</span>
            </div>
            <div class="status-item">
                <span class="dot yellow"></span>
                <span>盈利</span>
                <span class="status-value" id="displayProfit">1,877.94</span>
            </div>
            <div class="status-item">
                <span>运行资金</span>
                <span class="status-value" id="currentFundUsage">1,000-2,000</span>
            </div>
            <button class="action-btn profit" id="profitDetailBtn">📋 盈利</button>
            <button class="action-btn danger" id="clearLogBtn">清空</button>
            <button class="action-btn" id="pauseBtn">⏸️ 暂停</button>
            <button class="action-btn" id="resumeBtn" style="display:none; border-color: rgba(100,255,180,0.4); color: #6effb0;">▶️ 继续</button>
        </div>
    </div>

    <!-- 日志区域 -->
    <div class="log-area" id="logArea"></div>
</div>

<!-- Profit Modal -->
<div id="profitModal" class="modal">
    <div class="modal-content">
        <div class="modal-header">
            <h3>📈 盈利记录</h3>
            <button class="modal-close" id="closeModalBtn">✕</button>
        </div>
        <div class="profit-list" id="profitList">
            <div class="empty-profit">暂无盈利记录</div>
        </div>
        <div class="modal-footer">
            总计: <span id="modalTotalProfit">0.00</span> USDT
        </div>
    </div>
</div>

<script>
    // DOM Elements
    const displayBalanceSpan = document.getElementById('displayBalance');
    const displayProfitSpan = document.getElementById('displayProfit');
    const currentFundUsageSpan = document.getElementById('currentFundUsage');
    const logArea = document.getElementById('logArea');
    const runningTimeSpan = document.getElementById('runningTime');
    const statusText = document.getElementById('statusText');
    
    const balanceInput = document.getElementById('balanceInput');
    const profitInput = document.getElementById('profitInput');
    const updateBtn = document.getElementById('updateBtn');
    
    const clearLogBtn = document.getElementById('clearLogBtn');
    const profitDetailBtn = document.getElementById('profitDetailBtn');
    const profitModal = document.getElementById('profitModal');
    const closeModalBtn = document.getElementById('closeModalBtn');
    const profitListDiv = document.getElementById('profitList');
    const modalTotalProfitSpan = document.getElementById('modalTotalProfit');
    const pauseBtn = document.getElementById('pauseBtn');
    const resumeBtn = document.getElementById('resumeBtn');

    const MIN_FUND = 1000;
    const MAX_FUND = 2000;

    const chains = ['Arbitrum', 'Optimism', 'Base', 'BSC', 'Polygon', 'Avalanche', 'zkSync Era', 'Linea', 'Scroll', 'Mantle'];
    const tokens = ['ETH', 'USDC', 'USDT', 'WBTC', 'LINK', 'UNI', 'ARB', 'OP', 'SOL', 'BNB', 'PEPE', 'WIF', 'BONK', 'DOGE', 'MATIC', 'AVAX', 'SUI', 'ORDI', 'XRP', 'LTC'];
    
    const gasMessages = [
        '⛽ Gas: 24 Gwei · 网络流畅',
        '⛽ Gas 波动: 31 Gwei · 等待确认',
        '⛽ 拥堵 47 Gwei · 延迟执行',
        '⛽ Layer2 Gas: 0.08 Gwei · 极低',
        '⛽ Gas 飙升 55 Gwei · 暂缓'
    ];
    
    const scanMessages = [
        '🔍 扫描 {chain} 全链流动性 · 监控价差',
        '📡 获取 {chain} 价格预言机数据',
        '🔄 对比 {chain} 交易对深度',
        '📊 分析 {chain} 滑点模型'
    ];
    
    const statusMessages = [
        '🧠 AI 计算跨链套利概率',
        '📈 捕获最优价差信号',
        '⚡ 内存池监听 MEV 机会',
        '🔗 多链 RPC 同步良好',
        '✅ 引擎心跳正常'
    ];

    let logInterval = null;
    let globalClockInterval = null;
    let currentBalance = 1988.00;
    let currentProfitValue = 1877.94;
    let activeArbitrage = null;
    let arbitrageTimeoutId = null;
    let profitRecords = [];
    let isPaused = false;

    function isFundInRange() { return currentBalance >= MIN_FUND && currentBalance <= MAX_FUND; }
    
    function updateFundUsageDisplay() {
        if (currentFundUsageSpan) {
            const inRange = isFundInRange();
            currentFundUsageSpan.textContent = inRange ? `${currentBalance.toFixed(0)} USDT` : `⚠️ ${currentBalance.toFixed(0)} USDT`;
            currentFundUsageSpan.style.color = inRange ? '#a78aff' : '#ff7a7a';
        }
    }

    function getCurrentTimestamp() {
        const now = new Date();
        const Y = now.getFullYear(), M = String(now.getMonth()+1).padStart(2,'0'), D = String(now.getDate()).padStart(2,'0');
        const hh = String(now.getHours()).padStart(2,'0'), mm = String(now.getMinutes()).padStart(2,'0'), ss = String(now.getSeconds()).padStart(2,'0');
        return `${Y}-${M}-${D} ${hh}:${mm}:${ss}`;
    }

    function updateClock() {
        const full = getCurrentTimestamp();
        if (runningTimeSpan) runningTimeSpan.innerText = full;
    }

    function syncDisplayStats() {
        if (displayBalanceSpan) displayBalanceSpan.textContent = currentBalance.toFixed(2);
        if (displayProfitSpan) displayProfitSpan.textContent = currentProfitValue.toFixed(2);
        balanceInput.value = currentBalance.toFixed(2);
        profitInput.value = currentProfitValue.toFixed(2);
        updateFundUsageDisplay();
    }

    function randomChain() { return chains[Math.floor(Math.random() * chains.length)]; }
    function randomToken() { return tokens[Math.floor(Math.random() * tokens.length)]; }
    function getSpreadPercent() { return (0.08 + Math.random() * 0.20).toFixed(3); }
    function getArbProfitAmount() { return 0.15 + Math.random() * 1.85; }

    function generateArbitrageOpportunity() {
        let chainA = randomChain(), chainB = randomChain();
        while (chainB === chainA) chainB = randomChain();
        const token = randomToken(), spread = getSpreadPercent(), profit = getArbProfitAmount();
        const msg = `🎯 检测到 ${token} 在 ${chainA} ⇄ ${chainB} 链价差 ${spread}% · 执行套利搬砖`;
        return {
            srcChain: chainA, dstChain: chainB, token, spreadPercent: spread, profitUsdt: profit,
            message: msg
        };
    }

    function addProfitRecord(arb, gain) {
        profitRecords.unshift({ 
            time: getCurrentTimestamp(), 
            amount: gain, 
            token: arb.token, 
            chain1: arb.srcChain, 
            chain2: arb.dstChain, 
            spread: arb.spreadPercent
        });
        if (profitRecords.length > 50) profitRecords.pop();
    }

    function refreshProfitModal() {
        if (!profitListDiv) return;
        profitListDiv.innerHTML = '';
        if (profitRecords.length === 0) {
            profitListDiv.innerHTML = '<div class="empty-profit">暂无盈利记录</div>';
            modalTotalProfitSpan.textContent = '0.00';
            return;
        }
        let total = 0;
        profitRecords.forEach(rec => {
            total += rec.amount;
            const div = document.createElement('div');
            div.className = 'profit-record';
            div.innerHTML = `
                <div class="record-time">⏱️ ${rec.time}</div>
                <div class="record-detail">
                    <span>💰 ${rec.token} 跨链套利</span>
                    <span class="record-amount">+${rec.amount.toFixed(2)} USDT</span>
                </div>
                <div style="font-size:11px;color:#6f6299;margin-top:4px;">${rec.chain1} ⇄ ${rec.chain2} · 价差 ${rec.spread}%</div>
            `;
            profitListDiv.appendChild(div);
        });
        modalTotalProfitSpan.textContent = total.toFixed(2);
    }

    function executeSuccessArbitrage(arb) {
        if (!arb) return;
        if (!isFundInRange()) {
            const tp = getCurrentTimestamp().slice(11);
            const w = document.createElement('div');
            w.className = 'log-line';
            w.innerHTML = `<span class="log-time">${tp}</span> <span class="log-warning">⚠️ 运行资金 ${currentBalance.toFixed(2)} USDT 超出范围 · 套利暂停</span>`;
            if (logArea) { logArea.appendChild(w); logArea.scrollTop = logArea.scrollHeight; trimLogs(); }
            activeArbitrage = null; if (arbitrageTimeoutId) clearTimeout(arbitrageTimeoutId); arbitrageTimeoutId = null;
            return;
        }
        const gain = arb.profitUsdt;
        currentBalance += gain; currentProfitValue += gain;
        syncDisplayStats();
        addProfitRecord(arb, gain);
        const tp = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${tp}</span> <span class="log-success">✅ 已成功套利 ${gain.toFixed(2)} USDT 请在OKX钱包账单查看详情</span>`;
        if (logArea) { logArea.appendChild(d); logArea.scrollTop = logArea.scrollHeight; trimLogs(); }
        activeArbitrage = null; if (arbitrageTimeoutId) clearTimeout(arbitrageTimeoutId); arbitrageTimeoutId = null;
    }

    function handleNewArbitrage(arb) {
        if (!logArea) return;
        if (!isFundInRange()) {
            const tm = getCurrentTimestamp().slice(11);
            const w = document.createElement('div');
            w.className = 'log-line';
            w.innerHTML = `<span class="log-time">${tm}</span> <span class="log-warning">⚠️ 机会忽略 · 运行资金 ${currentBalance.toFixed(2)} USDT 超出范围</span>`;
            logArea.appendChild(w); logArea.scrollTop = logArea.scrollHeight; trimLogs();
            return;
        }
        const tm = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${tm}</span> <span class="log-arbitrage">📣 ${arb.message}</span>`;
        logArea.appendChild(d); logArea.scrollTop = logArea.scrollHeight; trimLogs();
        if (activeArbitrage !== null) {
            const l = document.createElement('div');
            l.className = 'log-line';
            l.innerHTML = `<span class="log-time">${getCurrentTimestamp().slice(11)}</span> <span class="log-gas">⚠️ 另一套利进行中 · 机会被抢先</span>`;
            logArea.appendChild(l); logArea.scrollTop = logArea.scrollHeight; trimLogs();
            return;
        }
        activeArbitrage = arb;
        const delay = 6000 + Math.random() * 10000;
        if (arbitrageTimeoutId) clearTimeout(arbitrageTimeoutId);
        arbitrageTimeoutId = setTimeout(() => { if (activeArbitrage && !isPaused) executeSuccessArbitrage(activeArbitrage); arbitrageTimeoutId = null; }, delay);
    }

    function trimLogs() { if (!logArea) return; while (logArea.children.length > 120) logArea.removeChild(logArea.firstChild); }

    function generateOrdinaryLog() {
        const r = Math.random();
        let msg = '', cls = 'log-scan';
        if (r < 0.30) { msg = gasMessages[Math.floor(Math.random() * gasMessages.length)]; cls = 'log-gas'; }
        else if (r < 0.70) { 
            const c = randomChain(); 
            const template = scanMessages[Math.floor(Math.random() * scanMessages.length)];
            msg = template.replace('{chain}', c);
            cls = 'log-scan'; 
        }
        else { msg = statusMessages[Math.floor(Math.random() * statusMessages.length)]; cls = 'log-scan'; }
        if (Math.random() < 0.10) msg = `🔁 跨链路由更新: ${randomChain()} → ${randomChain()}`;
        return { msg, type: cls };
    }

    function addQuantLog() {
        if (isPaused) return;
        if (!logArea) return;
        let chance = 0.05;
        if (activeArbitrage !== null) chance = 0.015;
        if (Math.random() < chance && activeArbitrage === null) {
            const arb = generateArbitrageOpportunity();
            handleNewArbitrage(arb);
            return;
        }
        const ord = generateOrdinaryLog();
        const ts = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${ts}</span> <span class="${ord.type}">${ord.msg}</span>`;
        logArea.appendChild(d); logArea.scrollTop = logArea.scrollHeight; trimLogs();
    }

    function clearLogsHandler() {
        if (!logArea) return;
        logArea.innerHTML = '';
        const ts = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${ts}</span> <span class="log-info">🗑️ 日志已清空 · 引擎继续运行</span>`;
        logArea.appendChild(d);
    }

    function initRunningLogs() {
        if (!logArea) return;
        logArea.innerHTML = '';
        const ts = getCurrentTimestamp().slice(11);
        const msgs = [
            '✦ Nova 量化引擎 v3.0 已激活',
            '🌐 跨链聚合: Arbitrum/OP/Base/Polygon/zkSync 等 10 链',
            '📊 运行资金: 1000–2000 USDT · 合规检测启用',
            '⏳ 实时监控链上价差 · 等待套利信号...'
        ];
        msgs.forEach(m => {
            const d = document.createElement('div');
            d.className = 'log-line';
            d.innerHTML = `<span class="log-time">${ts}</span> <span class="log-info">✦ ${m}</span>`;
            logArea.appendChild(d);
        });
    }

    function startQuantEngine() {
        if (logInterval) clearInterval(logInterval);
        initRunningLogs();
        logInterval = setInterval(addQuantLog, 6000 + Math.random() * 9000);
    }

    function stopQuantEngine() {
        if (logInterval) { clearInterval(logInterval); logInterval = null; }
        if (arbitrageTimeoutId) { clearTimeout(arbitrageTimeoutId); arbitrageTimeoutId = null; }
        activeArbitrage = null;
    }

    function startGlobalClock() {
        if (globalClockInterval) clearInterval(globalClockInterval);
        updateClock();
        globalClockInterval = setInterval(updateClock, 1000);
    }

    updateBtn.addEventListener('click', function() {
        let nb = parseFloat(balanceInput.value);
        let np = parseFloat(profitInput.value);
        if (isNaN(nb) || nb < 0) nb = 1500.00;
        if (isNaN(np)) np = 0.00;
        currentBalance = nb;
        currentProfitValue = np;
        syncDisplayStats();
        const ts = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${ts}</span> <span class="log-info">📊 手动更新: 余额=${nb.toFixed(2)} USDT, 盈利=${np.toFixed(2)} USDT</span>`;
        logArea.appendChild(d); logArea.scrollTop = logArea.scrollHeight; trimLogs();
    });

    pauseBtn.addEventListener('click', function() {
        isPaused = true;
        this.style.display = 'none';
        resumeBtn.style.display = 'inline-block';
        statusText.textContent = '已暂停';
        statusText.style.color = '#ffaa66';
        const ts = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${ts}</span> <span class="log-warning">⏸️ 引擎已暂停 · 等待恢复</span>`;
        logArea.appendChild(d); logArea.scrollTop = logArea.scrollHeight;
    });

    resumeBtn.addEventListener('click', function() {
        isPaused = false;
        this.style.display = 'none';
        pauseBtn.style.display = 'inline-block';
        statusText.textContent = '运行中';
        statusText.style.color = '#6effb0';
        const ts = getCurrentTimestamp().slice(11);
        const d = document.createElement('div');
        d.className = 'log-line';
        d.innerHTML = `<span class="log-time">${ts}</span> <span class="log-execute">▶️ 引擎已恢复 · 继续监控</span>`;
        logArea.appendChild(d); logArea.scrollTop = logArea.scrollHeight;
    });

    if (clearLogBtn) clearLogBtn.onclick = clearLogsHandler;
    if (profitDetailBtn) profitDetailBtn.onclick = () => { refreshProfitModal(); profitModal.style.display = 'flex'; };
    if (closeModalBtn) closeModalBtn.onclick = () => { profitModal.style.display = 'none'; };
    window.onclick = (e) => { if (e.target === profitModal) profitModal.style.display = 'none'; };

    syncDisplayStats();
    startQuantEngine();
    startGlobalClock();
    updateFundUsageDisplay();

    window.addEventListener('beforeunload', () => { 
        if (logInterval) clearInterval(logInterval); 
        if (arbitrageTimeoutId) clearTimeout(arbitrageTimeoutId); 
    });
</script>
</body>
</html>
