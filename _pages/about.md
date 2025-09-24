<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>高性能方块墙 1000x1000</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }
        
        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #1a2a6c, #b21f1f, #fdbb2d);
            padding: 20px;
            color: #fff;
            overflow: hidden;
        }
        
        .header {
            text-align: center;
            margin-bottom: 20px;
            width: 100%;
            max-width: 1200px;
            background: rgba(0, 0, 0, 0.6);
            padding: 20px;
            border-radius: 15px;
            box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
        }
        
        h1 {
            font-size: 2.8rem;
            margin-bottom: 10px;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            color: #ffcc00;
        }
        
        .description {
            font-size: 1.2rem;
            opacity: 0.9;
            max-width: 800px;
            margin: 0 auto;
            line-height: 1.6;
        }
        
        .controls {
            display: flex;
            gap: 20px;
            margin: 20px 0;
            flex-wrap: wrap;
            justify-content: center;
            background: rgba(0, 0, 0, 0.5);
            padding: 15px;
            border-radius: 10px;
            width: 100%;
            max-width: 1200px;
        }
        
        .control-group {
            display: flex;
            flex-direction: column;
            gap: 8px;
            min-width: 200px;
        }
        
        label {
            font-weight: 500;
            font-size: 1.1rem;
        }
        
        input[type="range"] {
            width: 100%;
        }
        
        .color-preview {
            width: 30px;
            height: 30px;
            border-radius: 6px;
            border: 2px solid white;
            margin: 0 auto;
        }
        
        .color-palette {
            display: flex;
            flex-wrap: wrap;
            gap: 8px;
            margin-top: 8px;
        }
        
        .color-option {
            width: 30px;
            height: 30px;
            border-radius: 6px;
            border: 2px solid transparent;
            cursor: pointer;
            transition: transform 0.2s, border-color 0.2s;
        }
        
        .color-option:hover {
            transform: scale(1.1);
        }
        
        .color-option.selected {
            border-color: white;
            transform: scale(1.1);
        }
        
        .viewport-container {
            position: relative;
            width: 100%;
            height: 60vh;
            max-width: 1200px;
            overflow: hidden;
            border: 3px solid rgba(255, 255, 255, 0.2);
            border-radius: 10px;
            background: white; /* 背景改为白色 */
            box-shadow: 0 12px 40px rgba(0, 0, 0, 0.4);
        }
        
        canvas {
            position: absolute;
            top: 0;
            left: 0;
            cursor: pointer;
        }
        
        .footer {
            margin-top: 20px;
            text-align: center;
            opacity: 0.8;
            font-size: 0.9rem;
            background: rgba(0, 0, 0, 0.5);
            padding: 10px 20px;
            border-radius: 10px;
            max-width: 1200px;
            width: 100%;
        }
        
        .stats {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 15px;
            font-size: 0.9rem;
        }
        
        .stat {
            background: rgba(255, 255, 255, 0.1);
            padding: 8px 15px;
            border-radius: 8px;
        }
        
        .loading {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            font-size: 1.5rem;
            background: rgba(0, 0, 0, 0.7);
            padding: 20px 40px;
            border-radius: 10px;
            z-index: 10;
            color: white;
        }
        
        /* 限制提示样式 */
        .limit-info {
            background: rgba(255, 100, 100, 0.3);
            padding: 10px;
            border-radius: 8px;
            margin-top: 10px;
            text-align: center;
            font-size: 0.9rem;
            transition: all 0.3s;
        }
        
        .limit-info.limited {
            background: rgba(255, 50, 50, 0.7);
            animation: pulse 1s infinite;
        }
        
        @keyframes pulse {
            0% { transform: scale(1); }
            50% { transform: scale(1.02); }
            100% { transform: scale(1); }
        }
        
        .cooldown-bar {
            width: 100%;
            height: 6px;
            background: rgba(255, 255, 255, 0.2);
            border-radius: 3px;
            margin-top: 5px;
            overflow: hidden;
        }
        
        .cooldown-progress {
            height: 100%;
            background: linear-gradient(90deg, #4cd964, #ffcc00);
            border-radius: 3px;
            transition: width 0.3s;
            width: 0%;
        }
        
        @media (max-width: 768px) {
            .controls {
                flex-direction: column;
                align-items: center;
            }
            
            .control-group {
                width: 100%;
            }
            
            h1 {
                font-size: 2rem;
            }
            
            .description {
                font-size: 1rem;
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <h1>高性能方块墙 1000x1000</h1>
        <p class="description">使用Canvas技术优化的百万方块墙，提供流畅的交互体验。使用缩放控制查看细节，选择颜色并点击方块进行绘制。</p>
    </div>
    
    <div class="controls">
        <div class="control-group">
            <label for="zoom">缩放控制: <span id="zoomValue">100%</span></label>
            <input type="range" id="zoom" min="10" max="500" value="100">
        </div>
        
        <div class="control-group">
            <label>选择颜色:</label>
            <div class="color-palette" id="colorPalette">
                <div class="color-option selected" style="background-color: #ff3b30;" data-color="#ff3b30"></div>
                <div class="color-option" style="background-color: #ff9500;" data-color="#ff9500"></div>
                <div class="color-option" style="background-color: #ffcc00;" data-color="#ffcc00"></div>
                <div class="color-option" style="background-color: #4cd964;" data-color="#4cd964"></div>
                <div class="color-option" style="background-color: #007aff;" data-color="#007aff"></div>
                <div class="color-option" style="background-color: #5856d6;" data-color="#5856d6"></div>
                <div class="color-option" style="background-color: #ff2d55;" data-color="#ff2d55"></div>
                <div class="color-option" style="background-color: #000000;" data-color="#000000"></div>
            </div>
        </div>
        
        <div class="control-group">
            <label>当前颜色:</label>
            <div class="color-preview" id="colorPreview" style="background-color: #ff3b30;"></div>
        </div>
        
        <!-- 新增绘制限制信息 -->
        <div class="control-group">
            <label>绘制限制:</label>
            <div class="limit-info" id="limitInfo">
                <div>每分钟最多绘制10个像素</div>
                <div>剩余: <span id="remainingDraws">10</span>/10</div>
                <div class="cooldown-bar">
                    <div class="cooldown-progress" id="cooldownProgress"></div>
                </div>
                <div id="cooldownText">冷却时间: 0秒</div>
            </div>
        </div>
    </div>
    
    <div class="viewport-container" id="viewportContainer">
        <div class="loading" id="loading">初始化中...</div>
        <canvas id="gridCanvas"></canvas>
    </div>
    
    <div class="footer">
        <p>使用鼠标滚轮可以放大缩小，点击并拖拽可以移动视图</p>
        <div class="stats">
            <div class="stat">方块总数: 1,000,000</div>
            <div class="stat">已着色: <span id="coloredCount">0</span></div>
            <div class="stat">缩放级别: <span id="zoomLevel">1.0</span>x</div>
            <div class="stat">视口方块: <span id="visibleBlocks">0</span></div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', function() {
            const canvas = document.getElementById('gridCanvas');
            const ctx = canvas.getContext('2d');
            const container = document.getElementById('viewportContainer');
            const colorPreview = document.getElementById('colorPreview');
            const zoomSlider = document.getElementById('zoom');
            const zoomValue = document.getElementById('zoomValue');
            const zoomLevel = document.getElementById('zoomLevel');
            const coloredCountElement = document.getElementById('coloredCount');
            const visibleBlocksElement = document.getElementById('visibleBlocks');
            const loadingElement = document.getElementById('loading');
            const colorPalette = document.getElementById('colorPalette');
            
            // 新增：绘制限制相关元素
            const limitInfo = document.getElementById('limitInfo');
            const remainingDrawsElement = document.getElementById('remainingDraws');
            const cooldownProgress = document.getElementById('cooldownProgress');
            const cooldownText = document.getElementById('cooldownText');
            
            // 网格配置
            const GRID_SIZE = 1000;
            const BLOCK_SIZE = 2; // 每个方块的像素大小
            const BASE_GRID_SIZE = GRID_SIZE * BLOCK_SIZE;
            
            // 状态变量
            let currentColor = '#ff3b30';
            let isMouseDown = false;
            let startX, startY;
            let offsetX = 0, offsetY = 0;
            let scale = 1.0;
            let coloredCount = 0;
            let isDragging = false; // 新增：用于判断是否正在拖拽
            
            // 新增：绘制限制相关变量
            const MAX_DRAWS_PER_MINUTE = 10; // 每分钟最多绘制10个像素
            let drawTimestamps = []; // 存储每次绘制的时间戳
            let cooldownInterval = null; // 冷却时间计时器
            
            // 颜色数据存储 (使用Uint32Array优化内存使用)
            let colorData = new Array(GRID_SIZE * GRID_SIZE);
            
            // 初始化颜色数据 - 改为白色背景
            function initializeColorData() {
                for (let i = 0; i < GRID_SIZE * GRID_SIZE; i++) {
                    colorData[i] = 0xFFFFFF; // 默认颜色改为白色 #FFFFFF
                }
            }
            
            // 新增：检查绘制限制
            function checkDrawLimit() {
                const now = Date.now();
                const oneMinuteAgo = now - 60000; // 1分钟前的时间戳
                
                // 过滤掉超过1分钟的时间戳
                drawTimestamps = drawTimestamps.filter(timestamp => timestamp > oneMinuteAgo);
                
                // 检查是否达到限制
                return drawTimestamps.length < MAX_DRAWS_PER_MINUTE;
            }
            
            // 新增：记录绘制时间
            function recordDraw() {
                const now = Date.now();
                drawTimestamps.push(now);
                
                // 更新剩余绘制次数显示
                updateDrawLimitDisplay();
                
                // 启动冷却时间更新
                if (!cooldownInterval) {
                    cooldownInterval = setInterval(updateDrawLimitDisplay, 1000);
                }
            }
            
            // 新增：更新绘制限制显示
            function updateDrawLimitDisplay() {
                const now = Date.now();
                const oneMinuteAgo = now - 60000;
                
                // 过滤掉超过1分钟的时间戳
                drawTimestamps = drawTimestamps.filter(timestamp => timestamp > oneMinuteAgo);
                
                const remaining = MAX_DRAWS_PER_MINUTE - drawTimestamps.length;
                remainingDrawsElement.textContent = remaining;
                
                // 计算冷却时间（最早的时间戳+1分钟）
                if (drawTimestamps.length > 0) {
                    const earliestTimestamp = Math.min(...drawTimestamps);
                    const cooldownEnd = earliestTimestamp + 60000;
                    const secondsLeft = Math.max(0, Math.ceil((cooldownEnd - now) / 1000));
                    
                    if (secondsLeft > 0) {
                        cooldownText.textContent = `冷却时间: ${secondsLeft}秒`;
                        // 计算进度条百分比
                        const progress = ((60000 - (cooldownEnd - now)) / 60000) * 100;
                        cooldownProgress.style.width = `${progress}%`;
                        
                        // 如果达到限制，添加视觉提示
                        if (remaining <= 0) {
                            limitInfo.classList.add('limited');
                        } else {
                            limitInfo.classList.remove('limited');
                        }
                    } else {
                        cooldownText.textContent = '冷却时间: 0秒';
                        cooldownProgress.style.width = '0%';
                        limitInfo.classList.remove('limited');
                    }
                } else {
                    cooldownText.textContent = '冷却时间: 0秒';
                    cooldownProgress.style.width = '0%';
                    limitInfo.classList.remove('limited');
                }
                
                // 如果没有时间戳了，清除计时器
                if (drawTimestamps.length === 0 && cooldownInterval) {
                    clearInterval(cooldownInterval);
                    cooldownInterval = null;
                }
            }
            
            // 将hex颜色转换为数字
            function hexToRgb(hex) {
                const r = parseInt(hex.slice(1, 3), 16);
                const g = parseInt(hex.slice(3, 5), 16);
                const b = parseInt(hex.slice(5, 7), 16);
                return (r << 16) + (g << 8) + b;
            }
            
            // 将数字颜色转换为hex
            function rgbToHex(rgb) {
                const r = ((rgb >> 16) & 0xff).toString(16).padStart(2, '0');
                const g = ((rgb >> 8) & 0xff).toString(16).padStart(2, '0');
                const b = (rgb & 0xff).toString(16).padStart(2, '0');
                return `#${r}${g}${b}`;
            }
            
            // 调整Canvas大小
            function resizeCanvas() {
                canvas.width = container.clientWidth;
                canvas.height = container.clientHeight;
                renderViewport();
            }
            
            // 获取视口范围内的网格坐标
            function getViewportBounds() {
                const scaledBlockSize = BLOCK_SIZE * scale;
                const startCol = Math.max(0, Math.floor(offsetX / scaledBlockSize));
                const endCol = Math.min(GRID_SIZE - 1, Math.floor((offsetX + canvas.width) / scaledBlockSize));
                const startRow = Math.max(0, Math.floor(offsetY / scaledBlockSize));
                const endRow = Math.min(GRID_SIZE - 1, Math.floor((offsetY + canvas.height) / scaledBlockSize));
                
                return {
                    startCol, endCol, 
                    startRow, endRow,
                    count: (endCol - startCol + 1) * (endRow - startRow + 1)
                };
            }
            
            // 渲染视口内容
            function renderViewport() {
                const bounds = getViewportBounds();
                visibleBlocksElement.textContent = bounds.count.toLocaleString();
                
                ctx.clearRect(0, 0, canvas.width, canvas.height);
                
                const scaledBlockSize = BLOCK_SIZE * scale;
                
                for (let row = bounds.startRow; row <= bounds.endRow; row++) {
                    for (let col = bounds.startCol; col <= bounds.endCol; col++) {
                        const index = row * GRID_SIZE + col;
                        const colorValue = colorData[index];
                        
                        // 修改：只绘制非白色方块（白色背景不需要绘制）
                        if (colorValue !== 0xFFFFFF) {
                            const x = col * scaledBlockSize - offsetX;
                            const y = row * scaledBlockSize - offsetY;
                            
                            // 只绘制在视口内的方块
                            if (x + scaledBlockSize > 0 && y + scaledBlockSize > 0 && 
                                x < canvas.width && y < canvas.height) {
                                ctx.fillStyle = rgbToHex(colorValue);
                                ctx.fillRect(x, y, scaledBlockSize, scaledBlockSize);
                            }
                        }
                    }
                }
                
                // 绘制网格线（在较高缩放级别时）
                if (scaledBlockSize > 8) {
                    ctx.strokeStyle = 'rgba(0, 0, 0, 0.1)'; // 修改：网格线改为黑色半透明
                    ctx.lineWidth = 1;
                    
                    for (let row = bounds.startRow; row <= bounds.endRow; row++) {
                        for (let col = bounds.startCol; col <= bounds.endCol; col++) {
                            const x = col * scaledBlockSize - offsetX;
                            const y = row * scaledBlockSize - offsetY;
                            
                            if (x + scaledBlockSize > 0 && y + scaledBlockSize > 0 && 
                                x < canvas.width && y < canvas.height) {
                                ctx.strokeRect(x, y, scaledBlockSize, scaledBlockSize);
                            }
                        }
                    }
                }
            }
            
            // 绘制单个方块
            function drawBlock(col, row) {
                const scaledBlockSize = BLOCK_SIZE * scale;
                const x = col * scaledBlockSize - offsetX;
                const y = row * scaledBlockSize - offsetY;
                
                // 只绘制在视口内的方块
                if (x + scaledBlockSize > 0 && y + scaledBlockSize > 0 && 
                    x < canvas.width && y < canvas.height) {
                    const index = row * GRID_SIZE + col;
                    const colorValue = colorData[index];
                    
                    // 修改：只绘制非白色方块
                    if (colorValue !== 0xFFFFFF) {
                        ctx.fillStyle = rgbToHex(colorValue);
                        ctx.fillRect(x, y, scaledBlockSize, scaledBlockSize);
                    } else {
                        // 如果是白色，清除该区域
                        ctx.clearRect(x, y, scaledBlockSize, scaledBlockSize);
                    }
                    
                    if (scaledBlockSize > 8) {
                        ctx.strokeStyle = 'rgba(0, 0, 0, 0.1)'; // 修改：网格线改为黑色半透明
                        ctx.strokeRect(x, y, scaledBlockSize, scaledBlockSize);
                    }
                }
            }
            
            // 初始化颜色选择
            function initializeColorSelection() {
                // 设置初始颜色
                colorPreview.style.backgroundColor = currentColor;
                
                // 颜色选择器事件
                const colorOptions = colorPalette.querySelectorAll('.color-option');
                colorOptions.forEach(option => {
                    option.addEventListener('click', function() {
                        // 移除之前选中的颜色
                        colorOptions.forEach(opt => opt.classList.remove('selected'));
                        // 设置当前选中的颜色
                        this.classList.add('selected');
                        currentColor = this.getAttribute('data-color');
                        colorPreview.style.backgroundColor = currentColor;
                    });
                });
            }
            
            // 初始化缩放控制
            function initializeZoom() {
                zoomSlider.addEventListener('input', function() {
                    const zoomPercent = this.value;
                    zoomValue.textContent = `${zoomPercent}%`;
                    
                    // 计算缩放比例
                    scale = zoomPercent / 100;
                    zoomLevel.textContent = scale.toFixed(1);
                    
                    renderViewport();
                });
            }
            
            // 初始化拖拽功能
            function initializeDrag() {
                container.addEventListener('mousedown', (e) => {
                    isMouseDown = true;
                    isDragging = false; // 重置拖拽状态
                    startX = e.clientX - offsetX;
                    startY = e.clientY - offsetY;
                    container.style.cursor = 'grabbing';
                });
                
                container.addEventListener('mouseleave', () => {
                    isMouseDown = false;
                    container.style.cursor = 'default';
                });
                
                container.addEventListener('mouseup', () => {
                    isMouseDown = false;
                    container.style.cursor = 'default';
                });
                
                container.addEventListener('mousemove', (e) => {
                    if (!isMouseDown) return;
                    
                    // 标记为正在拖拽
                    isDragging = true;
                    
                    offsetX = e.clientX - startX;
                    offsetY = e.clientY - startY;
                    
                    // 限制偏移范围
                    const maxOffset = GRID_SIZE * BLOCK_SIZE * scale;
                    offsetX = Math.max(0, Math.min(offsetX, maxOffset - canvas.width));
                    offsetY = Math.max(0, Math.min(offsetY, maxOffset - canvas.height));
                    
                    renderViewport();
                });
                
                // 添加滚轮缩放
                container.addEventListener('wheel', (e) => {
                    e.preventDefault();
                    
                    // 计算缩放前的鼠标位置对应的网格位置
                    const mouseX = e.clientX + offsetX;
                    const mouseY = e.clientY + offsetY;
                    
                    const zoom = zoomSlider.value;
                    if (e.deltaY < 0) {
                        // 放大
                        zoomSlider.value = Math.min(500, parseInt(zoom) + 10);
                    } else {
                        // 缩小
                        zoomSlider.value = Math.max(10, parseInt(zoom) - 10);
                    }
                    
                    // 计算缩放后的偏移，使鼠标位置保持在同一网格位置
                    const newScale = zoomSlider.value / 100;
                    offsetX = mouseX - e.clientX * (newScale / scale);
                    offsetY = mouseY - e.clientY * (newScale / scale);
                    
                    scale = newScale;
                    
                    // 限制偏移范围
                    const maxOffset = GRID_SIZE * BLOCK_SIZE * scale;
                    offsetX = Math.max(0, Math.min(offsetX, maxOffset - canvas.width));
                    offsetY = Math.max(0, Math.min(offsetY, maxOffset - canvas.height));
                    
                    // 更新UI
                    zoomValue.textContent = `${zoomSlider.value}%`;
                    zoomLevel.textContent = scale.toFixed(1);
                    
                    renderViewport();
                });
            }
            
            // 初始化点击事件（添加绘制限制）
            function initializeClick() {
                canvas.addEventListener('click', (e) => {
                    // 如果是拖拽操作，不执行绘制
                    if (isDragging) {
                        isDragging = false; // 重置拖拽状态
                        return;
                    }
                    
                    // 检查绘制限制
                    if (!checkDrawLimit()) {
                        alert('绘制限制：每分钟最多绘制10个像素，请等待冷却时间结束。');
                        return;
                    }
                    
                    const rect = canvas.getBoundingClientRect();
                    const x = e.clientX - rect.left;
                    const y = e.clientY - rect.top;
                    
                    // 计算点击的网格坐标
                    const col = Math.floor((x + offsetX) / (BLOCK_SIZE * scale));
                    const row = Math.floor((y + offsetY) / (BLOCK_SIZE * scale));
                    
                    if (col >= 0 && col < GRID_SIZE && row >= 0 && row < GRID_SIZE) {
                        const index = row * GRID_SIZE + col;
                        const currentColorValue = hexToRgb(currentColor);
                        
                        // 检查是否已经着色（从白色改为其他颜色）
                        if (colorData[index] === 0xFFFFFF && currentColorValue !== 0xFFFFFF) {
                            coloredCount++;
                            coloredCountElement.textContent = coloredCount;
                        }
                        // 如果从其他颜色改回白色，减少计数
                        else if (colorData[index] !== 0xFFFFFF && currentColorValue === 0xFFFFFF) {
                            coloredCount--;
                            coloredCountElement.textContent = coloredCount;
                        }
                        
                        // 更新颜色数据
                        colorData[index] = currentColorValue;
                        
                        // 记录绘制时间
                        recordDraw();
                        
                        // 绘制更新的方块
                        drawBlock(col, row);
                    }
                });
            }
            
            // 初始化所有功能
            function initialize() {
                initializeColorData();
                resizeCanvas();
                initializeColorSelection();
                initializeZoom();
                initializeDrag();
                initializeClick();
                
                // 初始化绘制限制显示
                updateDrawLimitDisplay();
                
                // 监听窗口大小变化
                window.addEventListener('resize', resizeCanvas);
                
                // 隐藏加载提示
                loadingElement.style.display = 'none';
            }
            
            // 启动初始化
            initialize();
        });
    </script>
</body>
</html>
