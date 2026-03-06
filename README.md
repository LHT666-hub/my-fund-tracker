<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>🏆 我的资产排行榜 Pro</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.1/dist/chart.umd.min.js"></script>
    <style>
        :root { 
            --up-color: #ff4d4f; --down-color: #52c41a; 
            --glass-bg: rgba(255, 255, 255, 0.95);
            --gradient-start: #e0c3fc; --gradient-end: #8ec5fc;
        }

        body { 
            background: linear-gradient(135deg, var(--gradient-start) 0%, var(--gradient-end) 100%);
            min-height: 100vh; font-family: 'PingFang SC', sans-serif; margin: 0; padding: 20px; color: #333;
        }
        
        .page-header { text-align: center; margin-bottom: 35px; }
        .page-header h1 { font-size: 2.8rem; font-weight: 900; margin: 0; color: #1a2a3a; text-shadow: 0 4px 12px rgba(0,0,0,0.1); }
        .page-header .subtitle { font-size: 1.1rem; color: #4a4a4a; margin-top: 10px; font-weight: 500; }

        .summary-wrapper { 
            max-width: 1000px; margin: 0 auto 30px; background: var(--glass-bg); padding: 25px; border-radius: 24px; 
            box-shadow: 0 10px 30px rgba(0,0,0,0.12); display: grid; grid-template-columns: 1.2fr 1fr; gap: 40px; align-items: center;
        }
        .summary-left { display: grid; grid-template-columns: 1fr 1fr; gap: 20px; }
        .summary-item { text-align: center; }
        .summary-item .label { font-size: 0.9rem; color: #666; }
        .summary-item .value { font-size: 1.8rem; font-weight: 800; margin-top: 5px; }

        .pie-chart-container { 
            position: relative; height: 280px; display: flex; align-items: center; justify-content: center;
        }
        #assetPieChart { max-width: 100%; }

        @media (max-width: 768px) {
            .summary-wrapper { grid-template-columns: 1fr; gap: 20px; }
            .summary-left { grid-template-columns: 1fr 1fr; }
            .pie-chart-container { height: 240px; }
        }

        .add-fund-area { text-align: center; margin-bottom: 30px; }
        .add-fund-area input { padding: 12px 25px; border: none; border-radius: 25px; width: 200px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); outline: none; }
        .add-fund-area button { padding: 12px 30px; border-radius: 25px; border: none; background: #6a11cb; color: white; font-weight: bold; cursor: pointer; margin-left: 10px; }

        .grid-container { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 25px; max-width: 1400px; margin: 0 auto; }
        
        .fund-card { 
            background: var(--glass-bg); border-radius: 20px; padding: 20px; position: relative;
            box-shadow: 0 8px 20px rgba(0,0,0,0.06); border: 1px solid rgba(255,255,255,0.4);
        }

        .fund-main-info { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 15px; }
        .fund-name { font-weight: bold; font-size: 1.1rem; color: #2c3e50; line-height: 1.3; }
        .gszzl { font-size: 1.6rem; font-weight: 900; }

        .info-section { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-bottom: 15px; }
        .info-box { background: rgba(0,0,0,0.04); padding: 10px; border-radius: 12px; }
        .info-label { font-size: 11px; color: #888; margin-bottom: 3px; }
        .info-val { font-size: 14px; font-weight: 700; }

        .input-controls { border-top: 1px dashed #ddd; padding-top: 15px; margin-top: 5px; }
        .input-group { margin-bottom: 10px; }
        .input-group label { display: block; font-size: 11px; color: #94a3b8; margin-bottom: 4px; }
        .input-group input { width: 100%; padding: 8px 12px; border: 1px solid #ddd; border-radius: 8px; box-sizing: border-box; font-size: 13px; outline: none; }
        .input-group input:focus { border-color: #6a11cb; background: #fff; }

        .btn-save { width: 100%; padding: 10px; border-radius: 10px; border: none; background: #6a11cb; color: white; cursor: pointer; font-size: 13px; font-weight: bold; display: none; margin-top: 5px; }

        .del-button { position: absolute; top: 15px; right: 15px; color: #cbd5e1; cursor: pointer; border: none; background: none; font-size: 18px; }
        .up { color: var(--up-color); }
        .down { color: var(--down-color); }

        /* 赛道标签样式 */
        .tag-label {
            display: inline-block; font-size: 11px; padding: 4px 10px; border-radius: 12px;
            margin: 3px 3px 3px 0; background: #e6f7ff; color: #1890ff; font-weight: 500;
            cursor: pointer; transition: all 0.2s;
        }
        .tag-label:hover { background: #bae7ff; }

        .tag-container { margin-bottom: 12px; min-height: 30px; }
        .tag-edit-area { margin: 10px 0; padding: 10px; background: #fafafa; border-radius: 8px; display: none; }
        .tag-edit-input { 
            width: 100%; padding: 8px; border: 1px solid #ddd; border-radius: 6px; 
            box-sizing: border-box; font-size: 12px; margin-bottom: 8px;
        }
        .tag-preset-list { display: flex; flex-wrap: wrap; gap: 6px; margin-bottom: 8px; }
        .tag-preset-btn {
            padding: 4px 12px; border: 1px solid #ddd; background: white; border-radius: 12px;
            font-size: 12px; cursor: pointer; transition: all 0.2s;
        }
        .tag-preset-btn:hover { background: #e6f7ff; border-color: #1890ff; color: #1890ff; }
        .tag-edit-buttons { display: flex; gap: 8px; }
        .tag-edit-btn { flex: 1; padding: 6px; border: none; border-radius: 6px; font-size: 12px; cursor: pointer; }
        .tag-edit-btn-primary { background: #1890ff; color: white; }
        .tag-edit-btn-secondary { background: #ddd; }

        /* 赛道统计区域 */
        .track-stats-wrapper { 
            max-width: 1200px; margin: 20px auto 30px; background: var(--glass-bg); padding: 20px; 
            border-radius: 20px; box-shadow: 0 8px 20px rgba(0,0,0,0.06);
        }
        .track-stats-title { font-size: 14px; font-weight: bold; color: #333; margin-bottom: 12px; }
        .track-stats-container { display: flex; gap: 15px; flex-wrap: wrap; }
        .track-stat-item { 
            padding: 10px 15px; background: rgba(0,0,0,0.04); border-radius: 12px; 
            min-width: 150px; text-align: center; cursor: pointer; transition: all 0.2s;
            border: 2px solid transparent;
        }
        .track-stat-item:hover { background: rgba(0,0,0,0.06); }
        .track-stat-item.active { background: #e6f7ff; border-color: #1890ff; }
        .track-stat-item .stat-name { font-size: 13px; color: #666; margin-bottom: 5px; }
        .track-stat-item .stat-value { font-size: 16px; font-weight: 800; }

        /* 按钮通用样式 */
        .filter-toggle, .sort-toggle { 
            display: inline-block; padding: 6px 14px; background: #f0f2f5; border: 1px solid #ddd; 
            border-radius: 6px; font-size: 12px; cursor: pointer; margin-left: 10px; transition: all 0.2s;
            margin-bottom: 10px;
        }
        .filter-toggle:hover, .sort-toggle:hover { background: #e6e8eb; }
        
        /* 筛选按钮高亮颜色 */
        .filter-toggle.active { background: #1890ff; color: white; border-color: #1890ff; }
        
        /* 排序按钮高亮颜色（换个紫色系区分功能） */
        .sort-toggle.active { background: #9b59b6; color: white; border-color: #9b59b6; font-weight: bold;}
        
        .control-panel { text-align: center; margin-bottom: 20px; padding: 15px; background: var(--glass-bg); border-radius: 15px; max-width: 1400px; margin-left: auto; margin-right: auto;}
    </style>
</head>
<body>

<div class="page-header">
    <h1>🚀 我的基金实时追踪</h1>
    <div class="subtitle">洞悉每一刻的财富波动</div>
</div>

<div class="summary-wrapper">
    <div class="summary-left">
        <div class="summary-item">
            <div class="label">今日预估盈亏 (元)</div>
            <div id="total-day" class="value">0.00</div>
        </div>
        <div class="summary-item">
            <div class="label">资产总市值 (元)</div>
            <div id="total-market" class="value">0.00</div>
        </div>
    </div>
    <div class="pie-chart-container">
        <canvas id="assetPieChart"></canvas>
    </div>
</div>

<div class="add-fund-area">
    <input type="text" id="newCode" placeholder="输入6位代码">
    <button onclick="addFund()" style="margin-left: 10px;">追踪</button>
</div>

<div class="track-stats-wrapper" id="trackStatsWrapper" style="display: none;">
    <div class="track-stats-title">📊 赛道行情一览</div>
    <div class="track-stats-container" id="trackStatsContainer"></div>
</div>

<div class="control-panel">
    <div style="margin-bottom: 15px;">
        <span style="font-size: 13px; color: #666; font-weight: bold; margin-right: 5px;">🏷️ 赛道筛选:</span>
        <button class="filter-toggle" onclick="filterByTrack('all')">全部</button>
        <div id="filterButtons" style="display: inline-block;" data-currentFilter="all"></div>
    </div>
    
    <div style="border-top: 1px dashed #ddd; padding-top: 15px;">
        <span style="font-size: 13px; color: #666; font-weight: bold; margin-right: 5px;">⚡ 极速排序:</span>
        <button class="sort-toggle active" id="sort-default" onclick="changeSort('default')">默认顺序</button>
        <button class="sort-toggle" id="sort-dayProfit" onclick="changeSort('dayProfit')">今日盈亏</button>
        <button class="sort-toggle" id="sort-gszzl" onclick="changeSort('gszzl')">今日涨幅</button>
        <button class="sort-toggle" id="sort-amount" onclick="changeSort('amount')">仓位大小</button>
        <button class="sort-toggle" id="sort-profitRate" onclick="changeSort('profitRate')">总收益率</button>
    </div>
</div>

<div class="grid-container" id="fundGrid"></div>

<script>
    let myAssets = JSON.parse(localStorage.getItem('my_assets_v7')) || [];
    let fundStore = {};
    
    // 全局排序状态
    let currentSort = 'default';
    let sortDesc = true; // true = 降序, false = 升序

    async function fetchOne(code) {
        const url = `https://fundgz.1234567.com.cn/js/${code}.js?rt=${Date.now()}`;
        const proxy = `https://api.codetabs.com/v1/proxy?quest=${encodeURIComponent(url)}`;
        try {
            const res = await fetch(proxy);
            const text = await res.text();
            const match = text.match(/\((.*)\)/);
            if (match) {
                const newData = JSON.parse(match[1]);
                if (newData && newData.gszzl !== undefined) {
                    fundStore[code] = newData;
                    render();
                }
            }
        } catch (e) {}
    }

    // 切换排序逻辑
    function changeSort(type) {
        if (currentSort === type && type !== 'default') {
            sortDesc = !sortDesc; // 反转升降序
        } else {
            currentSort = type;
            sortDesc = true; // 切换新条件时，默认降序
        }

        // 更新按钮UI
        document.querySelectorAll('.sort-toggle').forEach(btn => {
            btn.classList.remove('active');
            // 清理原有箭头
            btn.innerText = btn.innerText.replace(' ↓', '').replace(' ↑', '');
            
            // 给当前选中的按钮加上箭头指示
            if (btn.id === 'sort-' + type && type !== 'default') {
                btn.innerText += (sortDesc ? ' ↓' : ' ↑');
            }
        });

        document.getElementById('sort-' + type).classList.add('active');
        render();
    }

    function render() {
        const grid = document.getElementById('fundGrid');
        grid.innerHTML = '';
        let totalDay = 0, totalMarket = 0;
        let fundList = [];  
        let trackStats = {};  

        const currentFilter = document.getElementById('filterButtons').dataset.currentFilter || 'all';

        // 1. 数据清洗与预计算 (把所有需要的数值提前算好)
        let enrichedAssets = myAssets.map(asset => {
            if (!asset.track) asset.track = [];
            const data = fundStore[asset.code];
            if (!data) return null;

            const gsz = parseFloat(data.gsz);
            const dwjz = parseFloat(data.dwjz);
            const gszzl = parseFloat(data.gszzl);
            const amount = asset.amount || 0;
            const share = amount / gsz;
            const dayProfit = (gsz - dwjz) * share;
            
            const costTotal = amount - (asset.profit || 0);
            const profitRate = costTotal > 0 ? (((asset.profit || 0) / costTotal) * 100) : 0;

            return { asset, data, gsz, dwjz, gszzl, share, dayProfit, costTotal, profitRate, amount };
        }).filter(item => item !== null);

        // 2. 统计所有赛道数据（不受后续过滤和排序影响）
        enrichedAssets.forEach(item => {
            item.asset.track.forEach(track => {
                if (!trackStats[track]) trackStats[track] = { profit: 0, amount: 0 };
                trackStats[track].profit += item.dayProfit;
                trackStats[track].amount += item.amount;
            });
        });

        // 3. 执行赛道筛选
        if (currentFilter !== 'all') {
            enrichedAssets = enrichedAssets.filter(item => item.asset.track.includes(currentFilter));
        }

        // 4. 执行极速排序逻辑
        if (currentSort !== 'default') {
            enrichedAssets.sort((a, b) => {
                let valA = a[currentSort];
                let valB = b[currentSort];
                
                // 处理可能是 NaN 的情况，当做 0 处理
                if (isNaN(valA)) valA = 0;
                if (isNaN(valB)) valB = 0;
                
                return sortDesc ? (valB - valA) : (valA - valB);
            });
        }

        // 5. 渲染卡片 HTML
        enrichedAssets.forEach(item => {
            const { asset, data, gsz, gszzl, dayProfit, costTotal, profitRate, amount } = item;

            totalDay += (isNaN(dayProfit) ? 0 : dayProfit);
            totalMarket += amount;

            fundList.push({ name: data.name, amount: amount });

            const isUp = gszzl >= 0;

            const tagHtml = (asset.track && asset.track.length > 0) ? 
                `<div class="tag-container">${asset.track.map(tag => `<span class="tag-label">${tag}</span>`).join('')}</div>` 
                : '';

            grid.innerHTML += `
                <div class="fund-card">
                    <button class="del-button" onclick="removeFund('${asset.code}')">✕</button>
                    <button class="del-button" style="right: 50px; color: #52c41a; font-size: 16px;" onclick="openTagEditor('${asset.code}')" title="编辑赛道">🏷️</button>
                    <div class="fund-main-info">
                        <div>
                            <div class="fund-name">${data.name}</div>
                            <div style="font-size:12px; color:#999">${data.fundcode}</div>
                        </div>
                        <div class="gszzl ${isUp?'up':'down'}">${isUp?'+':''}${gszzl}%</div>
                    </div>

                    ${tagHtml}

                    <div class="info-section">
                        <div class="info-box">
                            <div class="info-label">今日盈亏</div>
                            <div class="info-val ${dayProfit>=0?'up':'down'}">${dayProfit>=0?'+':''}${dayProfit.toFixed(2)}</div>
                        </div>
                        <div class="info-box">
                            <div class="info-label">持仓收益率</div>
                            <div class="info-val ${profitRate>=0?'up':'down'}">${profitRate>=0?'+':''}${profitRate.toFixed(2)}%</div>
                        </div>
                        <div class="info-box">
                            <div class="info-label">本金估算</div>
                            <div class="info-val">¥${costTotal.toFixed(2)}</div>
                        </div>
                        <div class="info-box">
                            <div class="info-label">当前净值</div>
                            <div class="info-val">${gsz}</div>
                        </div>
                    </div>

                    <div class="tag-edit-area" id="tag-edit-${asset.code}">
                        <input type="text" class="tag-edit-input" placeholder="如: AI人工智能" id="tag-input-${asset.code}">
                        <div class="tag-preset-list">
                            ${['AI人工智能', '机器人', '新能源', '宽基', '消费', '医药', '房地产', '科技'].map(tag => 
                                `<button class="tag-preset-btn" onclick="addTagToFund('${asset.code}', '${tag}')">${tag}</button>`
                            ).join('')}
                        </div>
                        <div class="tag-edit-buttons">
                            <button class="tag-edit-btn tag-edit-btn-primary" onclick="saveTagEdit('${asset.code}')">保存</button>
                            <button class="tag-edit-btn tag-edit-btn-secondary" onclick="closeTagEditor('${asset.code}')">取消</button>
                        </div>
                    </div>

                    <div class="input-controls">
                        <div class="input-group">
                            <label>持有总金额 (App显示的那个)</label>
                            <input type="number" id="amt-${asset.code}" value="${asset.amount||''}" oninput="showBtn('${asset.code}')" placeholder="如: 10000.50">
                        </div>
                        <div class="input-group">
                            <label>持有总收益 (App显示的那个)</label>
                            <input type="number" id="pro-${asset.code}" value="${asset.profit||''}" oninput="showBtn('${asset.code}')" placeholder="如: 500.00">
                        </div>
                        <button id="btn-${asset.code}" class="btn-save" onclick="saveAsset('${asset.code}')">确认修改并保存</button>
                    </div>
                </div>
            `;
        });

        document.getElementById('total-day').innerText = (totalDay >= 0 ? '+' : '') + totalDay.toFixed(2);
        document.getElementById('total-day').className = 'value ' + (totalDay >= 0 ? 'up' : 'down');
        document.getElementById('total-market').innerText = totalMarket.toFixed(2);
        document.title = `${totalDay >= 0 ? '+' : ''}${totalDay.toFixed(2)} | 资产实时看板`;
        
        updateTrackStats(trackStats);
        updatePieChart(fundList, totalMarket);
    }

    function showBtn(code) { document.getElementById(`btn-${code}`).style.display = 'block'; }

    function saveAsset(code) {
        const item = myAssets.find(a => a.code === code);
        if (item) {
            item.amount = parseFloat(document.getElementById(`amt-${code}`).value) || 0;
            item.profit = parseFloat(document.getElementById(`pro-${code}`).value) || 0;
            localStorage.setItem('my_assets_v7', JSON.stringify(myAssets));
            render();
        }
    }

    function addFund() {
        const code = document.getElementById('newCode').value.trim();
        if (/^\d{6}$/.test(code) && !myAssets.find(a => a.code === code)) {
            myAssets.push({ code, amount: 0, profit: 0, track: [] });
            localStorage.setItem('my_assets_v7', JSON.stringify(myAssets));
            fetchOne(code);
            document.getElementById('newCode').value = '';
        }
    }

    function removeFund(code) {
        if(confirm('不再追踪此基金？')){
            myAssets = myAssets.filter(a => a.code !== code);
            localStorage.setItem('my_assets_v7', JSON.stringify(myAssets));
            render();
        }
    }

    let pieChart = null;

    function updatePieChart(fundList, totalMarket) {
        const ctx = document.getElementById('assetPieChart');
        if (!ctx) return;

        if (fundList.length === 0) {
            if (pieChart) pieChart.destroy();
            ctx.style.opacity = '0.5';
            return;
        }

        ctx.style.opacity = '1';

        // 注意：饼图我们强制按照仓位大小降序排列，视觉上更清晰
        fundList.sort((a, b) => b.amount - a.amount);

        const labels = fundList.map((f, idx) => {
            const percent = totalMarket > 0 ? ((f.amount / totalMarket) * 100).toFixed(1) : 0;
            return `${f.name.substring(0, 6)}... ${percent}%`;
        });

        const data = fundList.map(f => f.amount);

        const colors = [
            '#FF6B6B', '#4ECDC4', '#45B7D1', '#FFA07A', '#98D8C8',
            '#F7DC6F', '#BB8FCE', '#85C1E2', '#F8B88B', '#A8E6CF'
        ];

        const backgroundColors = fundList.map((_, idx) => colors[idx % colors.length]);
        const borderColors = backgroundColors.map(color => color + 'dd');

        if (pieChart) {
            pieChart.data.labels = labels;
            pieChart.data.datasets[0].data = data;
            pieChart.data.datasets[0].backgroundColor = backgroundColors;
            pieChart.data.datasets[0].borderColor = borderColors;
            pieChart.update();
        } else {
            pieChart = new Chart(ctx, {
                type: 'doughnut',
                data: {
                    labels: labels,
                    datasets: [{
                        data: data,
                        backgroundColor: backgroundColors,
                        borderColor: borderColors,
                        borderWidth: 2,
                        hoverBorderWidth: 3
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    plugins: {
                        legend: { display: false },
                        tooltip: {
                            callbacks: {
                                label: function(context) { return `¥${context.parsed.toFixed(2)}`; }
                            },
                            backgroundColor: 'rgba(0, 0, 0, 0.8)',
                            padding: 12,
                            borderRadius: 8,
                            titleColor: '#fff',
                            bodyColor: '#fff'
                        }
                    }
                }
            });
        }
    }

    function openTagEditor(code) { document.getElementById(`tag-edit-${code}`).style.display = 'block'; }
    function closeTagEditor(code) { document.getElementById(`tag-edit-${code}`).style.display = 'none'; }
    function addTagToFund(code, tag) {
        const input = document.getElementById(`tag-input-${code}`);
        if (input.value) { input.value += ', ' + tag; } else { input.value = tag; }
    }

    function saveTagEdit(code) {
        const asset = myAssets.find(a => a.code === code);
        if (!asset) return;

        const input = document.getElementById(`tag-input-${code}`);
        const tags = input.value.split(',').map(t => t.trim()).filter(t => t);
        asset.track = tags;
        localStorage.setItem('my_assets_v7', JSON.stringify(myAssets));
        updateFilterButtons();
        render();
        closeTagEditor(code);
    }

    function updateFilterButtons() {
        const allTracks = new Set();
        myAssets.forEach(asset => {
            if (asset.track) { asset.track.forEach(track => allTracks.add(track)); }
        });

        const container = document.getElementById('filterButtons');
        container.innerHTML = Array.from(allTracks).map(track => 
            `<button class="filter-toggle" onclick="filterByTrack('${track}')">${track}</button>`
        ).join('');
    }

    function filterByTrack(track) {
        document.querySelectorAll('.filter-toggle').forEach(btn => btn.classList.remove('active'));
        
        const buttons = document.querySelectorAll('.filter-toggle');
        buttons.forEach(btn => {
            if (track === 'all' && btn.textContent === '全部') { btn.classList.add('active'); } 
            else if (track !== 'all' && btn.textContent === track) { btn.classList.add('active'); }
        });

        document.getElementById('filterButtons').dataset.currentFilter = track;
        render();
    }

    function updateTrackStats(trackStats) {
        const wrapper = document.getElementById('trackStatsWrapper');
        const container = document.getElementById('trackStatsContainer');

        if (Object.keys(trackStats).length === 0) {
            wrapper.style.display = 'none';
            return;
        }

        wrapper.style.display = 'block';
        let html = '';
        Object.entries(trackStats).forEach(([track, stats]) => {
            const isUp = stats.profit >= 0;
            html += `
                <div class="track-stat-item" onclick="filterByTrack('${track}')">
                    <div class="stat-name">${track}</div>
                    <div class="stat-value ${isUp ? 'up' : 'down'}">${isUp ? '+' : ''}${stats.profit.toFixed(2)}</div>
                    <div style="font-size: 11px; color: #999; margin-top: 3px;">¥${stats.amount.toFixed(2)}</div>
                </div>
            `;
        });
        container.innerHTML = html;
    }

    function init() {
        updateFilterButtons();
        setTimeout(() => {
            const allBtn = document.querySelector('.filter-toggle');
            if (allBtn) allBtn.classList.add('active');
        }, 0);
        myAssets.forEach(a => fetchOne(a.code));
        setInterval(() => myAssets.forEach(a => fetchOne(a.code)), 60000);
    }
    init();
</script>
</body>
</html>
