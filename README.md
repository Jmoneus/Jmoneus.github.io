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
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; }
        body { background-color: var(--bg-color); color: var(--text); padding: 10px; padding-bottom: 60px; }
        
        h2 { text-align: center; margin: 10px 0 15px 0; font-weight: 400; font-size: 1.2rem; letter-spacing: 1px; }

        /* Grid Wrapper for Max Width Constraint */
        .matrix-wrapper {
            max-width: 500px;
            margin: 0 auto;
        }

        /* Column Headers */
        .grid-headers {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
            text-align: center;
            margin-bottom: 8px;
            font-size: 0.9rem;
            font-weight: bold;
            text-transform: uppercase;
            color: var(--accent);
            letter-spacing: 0.5px;
        }

        /* 3x10 Grid Setup */
        .grid-container {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
        }

        .grid-cell {
            background: var(--grid-bg);
            border: 1px solid var(--cell-border);
            border-radius: 6px;
            min-height: 85px;
            padding: 6px;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            cursor: pointer;
            transition: border-color 0.2s;
            -webkit-tap-highlight-color: transparent;
        }

        .grid-cell:active { border-color: var(--accent); background: #252525; }

        /* Cell Preview Lines */
        .line-preview { font-size: 0.7rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis; height: 16px; }
        .line-t { color: #ff6b6b; border-bottom: 1px dashed #2a2a2a; }
        .line-m { color: #ffd166; border-bottom: 1px dashed #2a2a2a; }
        .line-b { color: #06d6a0; }
        .cell-label { font-size: 0.65rem; color: var(--text-muted); align-self: flex-end; margin-top: auto; font-weight: bold; }

        /* Modal / Edit Overlay */
        .modal {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.8); justify-content: center; align-items: center; z-index: 10;
        }

        .modal-content {
            background: #252525; padding: 20px; border-radius: 12px; width: 90%; max-width: 400px;
            border: 1px solid #444;
        }

        .modal-title { margin-bottom: 15px; font-size: 1rem; color: var(--accent); font-weight: bold; }

        .input-group { margin-bottom: 12px; display: flex; flex-direction: column; }
        .input-group label { font-size: 0.75rem; color: var(--text-muted); margin-bottom: 4px; text-transform: uppercase; font-weight: bold; }
        
        .input-group input {
            background: #111; border: 1px solid #444; color: #fff; padding: 10px;
            border-radius: 6px; font-size: 1rem; outline: none;
        }
        .input-group input:focus { border-color: var(--accent); }

        .btn-group { display: flex; gap: 10px; margin-top: 20px; }
        button {
            flex: 1; padding: 12px; border: none; border-radius: 6px; font-size: 1rem;
            cursor: pointer; font-weight: bold;
        }
        .btn-save { background: var(--accent); color: #fff; }
        .btn-cancel { background: #444; color: #fff; }

        /* Utilities */
        .clear-btn {
            display: block; width: 100%; max-width: 500px; margin: 20px auto 0 auto;
            background: #331c1c; color: #ff6b6b; border: 1px solid #552222;
        }
    </style>
</head>
<body>

    <h2>3 × 10 LOG MATRIX</h2>
    
    <div class="matrix-wrapper">
        <!-- Column Header Titles -->
        <div class="grid-headers">
            <div>Back</div>
            <div>Mid</div>
            <div>Front</div>
        </div>
        
        <!-- Main Grid -->
        <div class="grid-container" id="grid"></div>
        
        <button class="clear-btn" onclick="clearAllData()">Clear All Grid Data</button>
    </div>

    <!-- Input Modal -->
    <div class="modal" id="modal">
        <div class="modal-content">
            <div class="modal-title" id="modal-label">Editing Cell</div>
            
            <div class="input-group">
                <label>Top</label>
                <input type="text" id="input-top" autocomplete="off">
            </div>
            <div class="input-group">
                <label>Middle</label>
                <input type="text" id="input-mid" autocomplete="off">
            </div>
            <div class="input-group">
                <label>Bottom</label>
                <input type="text" id="input-bot" autocomplete="off">
            </div>

            <div class="btn-group">
                <button class="btn-cancel" onclick="closeModal()">Cancel</button>
                <button class="btn-save" onclick="saveCellData()">Save</button>
            </div>
        </div>
    </div>

    <script>
        const ROWS = 10;
        const COLS = 3;
        const colMap = { 1: 'B', 2: 'M', 3: 'F' };
        const colNames = { 1: 'Back', 2: 'Mid', 3: 'Front' };
        
        let gridData = JSON.parse(localStorage.getItem('gridLogData')) || {};
        let activeCellKey = null;

        const gridEl = document.getElementById('grid');
        const modalEl = document.getElementById('modal');

        // Generate Grid Layout
        for (let r = 1; r <= ROWS; r++) {
            for (let c = 1; c <= COLS; c++) {
                const cellKey = `${r}_${c}`;
                const cellData = gridData[cellKey] || { t: '', m: '', b: '' };
                const colLetter = colMap[c];

                const cell = document.createElement('div');
                cell.className = 'grid-cell';
                cell.id = `cell-${cellKey}`;
                cell.onclick = () => openModal(r, c);
                
                cell.innerHTML = `
                    <div class="line-preview line-t" id="p-t-${cellKey}">${cellData.t}</div>
                    <div class="line-preview line-m" id="p-m-${cellKey}">${cellData.m}</div>
                    <div class="line-preview line-b" id="p-b-${cellKey}">${cellData.b}</div>
                    <div class="cell-label">${colLetter}${r}</div>
                `;
                gridEl.appendChild(cell);
            }
        }

        function openModal(row, col) {
            activeCellKey = `${row}_${col}`;
            const colLetter = colMap[col];
            const colName = colNames[col];
            
            document.getElementById('modal-label').innerText = `Editing ${colName} — Cell ${colLetter}${row}`;
            
            const cellData = gridData[activeCellKey] || { t: '', m: '', b: '' };
            document.getElementById('input-top').value = cellData.t;
            document.getElementById('input-mid').value = cellData.m;
            document.getElementById('input-bot').value = cellData.b;
            
            modalEl.style.display = 'flex';
            document.getElementById('input-top').focus();
        }

        function closeModal() {
            modalEl.style.display = 'none';
            activeCellKey = null;
        }

        function saveCellData() {
            if (!activeCellKey) return;

            const tVal = document.getElementById('input-top').value;
            const mVal = document.getElementById('input-mid').value;
            const bVal = document.getElementById('input-bot').value;

            // Save to local storage state
            gridData[activeCellKey] = { t: tVal, m: mVal, b: bVal };
            localStorage.setItem('gridLogData', JSON.stringify(gridData));

            // Update UI Previews immediately
            document.getElementById(`p-t-${activeCellKey}`).innerText = tVal;
            document.getElementById(`p-m-${activeCellKey}`).innerText = mVal;
            document.getElementById(`p-b-${activeCellKey}`).innerText = bVal;

            closeModal();
        }

        function clearAllData() {
            if (confirm("Are you sure you want to wipe all matrix logs?")) {
                localStorage.removeItem('gridLogData');
                gridData = {};
                document.querySelectorAll('.line-preview').forEach(el => el.innerText = '');
            }
        }
    </script>
</body>
</html>
