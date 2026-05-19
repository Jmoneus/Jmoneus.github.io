<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Vehicle Grid Logger</title>
    <style>
        :root {
            --bg-color: #121212;
            --grid-bg: #1e1e1e;
            --cell-border: #333;
            --accent: #00adb5;
            --text: #eeeeee;
            --text-muted: #888;
            --sn-color: #888888;
            --panel-bg: #1b1b1b;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background-color: var(--bg-color); color: var(--text); padding: 20px 10px 60px 10px; }

        .matrix-wrapper { max-width: 500px; margin: 0 auto; }

        /* 4-Column Header Layout */
        .grid-headers {
            display: grid; grid-template-columns: 30px repeat(3, 1fr); gap: 8px;
            text-align: center; margin-bottom: 8px; font-size: 0.9rem; font-weight: bold;
            text-transform: uppercase; color: var(--accent); letter-spacing: 0.5px;
        }

        /* 4-Column Grid Setup */
        .grid-container { display: grid; grid-template-columns: 30px repeat(3, 1fr); gap: 8px; align-items: stretch; }
        .row-number-label { display: flex; align-items: center; justify-content: center; font-weight: bold; color: var(--text-muted); font-size: 0.95rem; }

        .grid-cell {
            background: var(--grid-bg); border: 1px solid var(--cell-border); border-radius: 6px;
            min-height: 90px; padding: 6px; display: flex; flex-direction: column;
            justify-content: space-between; cursor: pointer; transition: border-color 0.2s; -webkit-tap-highlight-color: transparent;
        }
        .grid-cell:active { border-color: var(--accent); background: #252525; }

        .line-preview { font-size: 0.7rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; height: 18px; line-height: 18px; }
        .line-t { color: #ff6b6b; border-bottom: 1px dashed #2a2a2a; }
        .line-m { color: #ffd166; border-bottom: 1px dashed #2a2a2a; }
        .line-b { color: #06d6a0; }
        
        .sn-preview { color: var(--sn-color); font-size: 0.65rem; font-weight: normal; margin-left: 2px; }
        .cell-label { font-size: 0.65rem; color: var(--text-muted); align-self: flex-end; margin-top: auto; font-weight: bold; }

        /* Share & Management Panel */
        .control-panel {
            background: var(--panel-bg); border: 1px solid #2d2d2d; border-radius: 8px;
            padding: 15px; margin-top: 25px;
        }
        .panel-title { font-size: 0.85rem; font-weight: bold; color: var(--accent); text-transform: uppercase; margin-bottom: 12px; letter-spacing: 0.5px; }

        /* Modals */
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.8); justify-content: center; align-items: center; z-index: 10; }
        .modal-content { background: #252525; padding: 20px; border-radius: 12px; width: 92%; max-width: 400px; border: 1px solid #444; max-height: 95vh; overflow-y: auto; }
        .modal-title { margin-bottom: 15px; font-size: 1rem; color: var(--accent); font-weight: bold; }

        .input-group { margin-bottom: 14px; display: flex; flex-direction: column; background: #1b1b1b; padding: 10px; border-radius: 6px; border: 1px solid #2d2d2d; }
        .input-group label { font-size: 0.75rem; color: var(--text-muted); margin-bottom: 5px; text-transform: uppercase; font-weight: bold; }
        .input-group input { background: #111; border: 1px solid #444; color: #fff; padding: 10px; border-radius: 6px; font-size: 1rem; outline: none; width: 100%; }
        .input-group input:focus { border-color: var(--accent); }
        .input-group input.sn-input { margin-top: 6px; background: #161616; border: 1px solid #333; font-size: 0.85rem; padding: 6px 10px; color: #b3b3b3; }

        .btn-group { display: flex; gap: 10px; margin-top: 10px; }
        button { flex: 1; padding: 12px; border: none; border-radius: 6px; font-size: 1rem; cursor: pointer; font-weight: bold; }
        
        .btn-save { background: var(--accent); color: #fff; }
        .btn-cancel { background: #444; color: #fff; }
        .btn-share { background: #1f6feb; color: #fff; }
        .btn-excel { background: #23a964; color: #fff; }
        .clear-btn { background: #331c1c; color: #ff6b6b; border: 1px solid #552222; margin-top: 15px; width: 100%; font-size: 0.9rem; padding: 10px; }
    </style>
</head>
<body>

    <div class="matrix-wrapper">
        <div class="grid-headers">
            <div></div><div>Back</div><div>Mid</div><div>Front</div>
        </div>
        
        <div class="grid-container" id="grid"></div>
        
        <!-- Share & Export Control Center -->
        <div class="control-panel">
            <div class="panel-title">Share & Data Exports</div>
            <div class="btn-group">
                <button class="btn-share" onclick="generateShareLink()">Copy Share Link</button>
                <button class="btn-excel" onclick="exportToCSV()">Export Spreadsheet</button>
            </div>
            <button class="clear-btn" onclick="clearAllData()">Wipe Local Grid Cache</button>
        </div>
    </div>

    <!-- Input Modal -->
    <div class="modal" id="modal">
        <div class="modal-content">
            <div class="modal-title" id="modal-label">Editing Cell</div>
            
            <div class="input-group" id="group-top">
                <label>Top</label>
                <input type="text" id="input-top" autocomplete="off" placeholder="Enter text..."><input type="text" id="input-top-sn" class="sn-input" autocomplete="off" placeholder="Serial Number / ID">
            </div>
            <div class="input-group">
                <label>Middle</label>
                <input type="text" id="input-mid" autocomplete="off" placeholder="Enter text..."><input type="text" id="input-mid-sn" class="sn-input" autocomplete="off" placeholder="Serial Number / ID">
            </div>
            <div class="input-group">
                <label>Bottom</label>
                <input type="text" id="input-bot" autocomplete="off" placeholder="Enter text..."><input type="text" id="input-bot-sn" class="sn-input" autocomplete="off" placeholder="Serial Number / ID">
            </div>

            <div class="btn-group">
                <button class="btn-cancel" onclick="closeModal()">Cancel</button>
                <button class="btn-save" onclick="saveCellData()">Save</button>
            </div>
        </div>
    </div>

    <script>
        const ROWS = 10; const COLS = 3;
        const colMap = { 1: 'B', 2: 'M', 3: 'F' };
        const colNames = { 1: 'Back', 2: 'Mid', 3: 'Front' };
        
        let gridData = {};

        // Parse payload values out of link if app initialized via Shared link
        const urlParams = new URLSearchParams(window.location.search);
        const sharedDataParam = urlParams.get('matrix');

        if (sharedDataParam) {
            try {
                gridData = JSON.parse(decodeURIComponent(atob(sharedDataParam)));
                localStorage.setItem('gridLogData', JSON.stringify(gridData));
                window.history.replaceState({}, document.title, window.location.pathname);
            } catch (e) {
                gridData = JSON.parse(localStorage.getItem('gridLogData')) || {};
            }
        } else {
            gridData = JSON.parse(localStorage.getItem('gridLogData')) || {};
        }

        const gridEl = document.getElementById('grid');
        const modalEl = document.getElementById('modal');

        function getCellHtml(text, sn) {
            if (!text && !sn) return '';
            return text ? `${text}${sn ? ` <span class="sn-preview">[${sn}]</span>` : ''}` : (sn ? `<span class="sn-preview">[${sn}]</span>` : '');
        }

        function renderGrid() {
            gridEl.innerHTML = '';
            for (let r = 1; r <= ROWS; r++) {
                const sideLabel = document.createElement('div');
                sideLabel.className = 'row-number-label'; sideLabel.innerText = r;
                gridEl.appendChild(sideLabel);

                for (let c = 1; c <= COLS; c++) {
                    const cellKey = `${r}_${c}`;
                    const d = gridData[cellKey] || { t: '', ts: '', m: '', ms: '', b: '', bs: '' };
                    const isBackColumn = (c === 1);

                    const cell = document.createElement('div');
                    cell.className = 'grid-cell'; cell.onclick = () => openModal(r, c);
                    cell.innerHTML = `
                        <div class="line-preview line-t" id="p-t-${cellKey}">${isBackColumn ? '' : getCellHtml(d.t, d.ts)}</div>
                        <div class="line-preview line-m" id="p-m-${cellKey}">${getCellHtml(d.m, d.ms)}</div>
                        <div class="line-preview line-b" id="p-b-${cellKey}">${getCellHtml(d.b, d.bs)}</div>
                        <div class="cell-label">${colMap[c]}${r}</div>
                    `;
                    gridEl.appendChild(cell);
                }
            }
        }

        function openModal(row, col) {
            activeCellKey = `${row}_${col}`;
            document.getElementById('modal-label').innerText = `Editing ${colNames[col]} — Cell ${colMap[col]}${row}`;
            document.getElementById('group-top').style.display = (col === 1) ? 'none' : 'flex';

            const d = gridData[activeCellKey] || { t: '', ts: '', m: '', ms: '', b: '', bs: '' };
            document.getElementById('input-top').value = d.t || ''; document.getElementById('input-top-sn').value = d.ts || '';
            document.getElementById('input-mid').value = d.m || ''; document.getElementById('input-mid-sn').value = d.ms || '';
            document.getElementById('input-bot').value = d.b || ''; document.getElementById('input-bot-sn').value = d.bs || '';
            
            modalEl.style.display = 'flex';
            if (col === 1) document.getElementById('input-mid').focus();
            else document.getElementById('input-top').focus();
        }

        function closeModal() { modalEl.style.display = 'none'; activeCellKey = null; }

        function saveCellData() {
            if (!activeCellKey) return;
            const isBackColumn = activeCellKey.endsWith('_1');

            gridData[activeCellKey] = {
                t: isBackColumn ? '' : document.getElementById('input-top').value,
                ts: isBackColumn ? '' : document.getElementById('input-top-sn').value,
                m: document.getElementById('input-mid').value,
                ms: document.getElementById('input-mid-sn').value,
                b: document.getElementById('input-bot').value,
                bs: document.getElementById('input-bot-sn').value
            };
            localStorage.setItem('gridLogData', JSON.stringify(gridData));
            renderGrid(); closeModal();
        }

        function generateShareLink() {
            try {
                const base64Data = btoa(encodeURIComponent(JSON.stringify(gridData)));
                const shareUrl = `${window.location.origin}${window.location.pathname}?matrix=${base64Data}`;
                navigator.clipboard.writeText(shareUrl).then(() => {
                    alert("Share Link Copied!");
                }).catch(() => {
                    prompt("Copy link manually:", shareUrl);
                });
            } catch(err) { alert("Error packing grid dataset."); }
        }

        // --- Transposed Clean Formatting CSV Export Logic ---
        function exportToCSV() {
            let csvContent = "data:text/csv;charset=utf-8,";
            
            // Header row labeling columns 1 through 10
            csvContent += "Position,1,2,3,4,5,6,7,8,9,10\r\n";

            // Internal formatting handler to bundle strings together cleanly inside cells
            function formatCell(text, sn) {
                if (!text && !sn) return "";
                let cellVal = text || "";
                if (sn) cellVal += ` [${sn}]`;
                return `"${cellVal.replace(/"/g, '""')}"`;
            }

            // Sections layout scheme matching your columns (1: Back, 2: Mid, 3: Front)
            const layoutMap = [
                { title: "Back Row", colIndex: 1 },
                { title: "Middle Row", colIndex: 2 },
                { title: "Front Row", colIndex: 3 }
            ];

            layoutMap.forEach(section => {
                // 1. Add Spacer row section title line
                csvContent += `"${section.title}",,,,,,,,,, \r\n`;

                // 2. Process Top Row data elements
                let topRow = ["top"];
                for (let r = 1; r <= ROWS; r++) {
                    const d = gridData[`${r}_${section.colIndex}`] || {};
                    // Leave completely empty if it's the Back column (since top text was removed)
                    topRow.push(section.colIndex === 1 ? "" : formatCell(d.t, d.ts));
                }
                csvContent += topRow.join(",") + "\r\n";

                // 3. Process Mid Row data elements
                let midRow = ["mid"];
                for (let r = 1; r <= ROWS; r++) {
                    const d = gridData[`${r}_${section.colIndex}`] || {};
                    midRow.push(formatCell(d.m, d.ms));
                }
                csvContent += midRow.join(",") + "\r\n";

                // 4. Process Bottom Row data elements
                let botRow = ["bottom"];
                for (let r = 1; r <= ROWS; r++) {
                    const d = gridData[`${r}_${section.colIndex}`] || {};
                    botRow.push(formatCell(d.b, d.bs));
                }
                csvContent += botRow.join(",") + "\r\n";
            });

            const encodedUri = encodeURI(csvContent);
            const link = document.createElement("a");
            link.setAttribute("href", encodedUri);
            link.setAttribute("download", `matrix_report_${new Date().toISOString().slice(0,10)}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        }

        function clearAllData() {
            if (confirm("Wipe local grid cache?")) {
                localStorage.removeItem('gridLogData'); gridData = {}; renderGrid();
            }
        }

        renderGrid();
    </script>
</body>
</html>
