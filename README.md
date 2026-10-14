<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>موقع تحليل الشطرنج الاحترافي</title>
    <!-- مكتبة Chessground للتصميم واللوحة -->
    <link rel="stylesheet" href="https://unpkg.com/chessground@9.1.0/assets/chessground.base.css">
    <link rel="stylesheet" href="https://unpkg.com/chessground@9.1.0/assets/chessground.brown.css">
    <link rel="stylesheet" href="https://unpkg.com/chessground@9.1.0/assets/chessground.cburnett.css">
    <style>
        body {
            font-family: Tahoma, sans-serif;
            background-color: #121212;
            color: #ffffff;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 { margin-bottom: 5px; color: #4CAF50; }
        .container {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            max-width: 1000px;
            width: 100%;
            margin-top: 20px;
            justify-content: center;
        }
        /* حجم رقعة الشطرنج */
        #board {
            width: 450px;
            height: 450px;
        }
        .sidebar {
            flex: 1;
            min-width: 300px;
            background: #1e1e1e;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 10px rgba(0,0,0,0.5);
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        textarea {
            width: 100%;
            height: 100px;
            background: #2b2b2b;
            color: #fff;
            border: 1px solid #444;
            border-radius: 4px;
            padding: 8px;
            resize: none;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px 15px;
            border-radius: 4px;
            cursor: pointer;
            font-weight: bold;
            font-size: 14px;
        }
        button:hover { background-color: #45a049; }
        #eval-bar {
            font-size: 18px;
            font-weight: bold;
            color: #ff9800;
        }
        .moves-box {
            background: #2b2b2b;
            height: 150px;
            overflow-y: auto;
            padding: 10px;
            border-radius: 4px;
            font-family: monospace;
        }
    </style>
</head>
<body>

    <h1>تحليل الشطرنج الاحترافي ♟️</h1>
    <p>حلل مبارياتك، اعرف النقلات، واكتشف أخطائك فوراً.</p>

    <div class="container">
        <!-- رقعة الشطرنج -->
        <div id="board"></div>

        <!-- اللوحة الجانبية للتحكم والتحليل -->
        <div class="sidebar">
            <h3>شريط التقييم والمحرك</h3>
            <div id="eval-bar">التقييم: متكافئ (0.0)</div>
            
            <h3>إدخال PGN / المباراة</h3>
            <textarea id="pgn-input" placeholder="ألصق كود الـ PGN هنا..."></textarea>
            <button onclick="loadPGN()">تحليل المباراة</button>

            <h3>سجل النقلات</h3>
            <div class="moves-box" id="moves-list">لا توجد نقلات بعد.</div>
        </div>
    </div>

    <!-- مكتبات Chess.js و Chessground عبر CDN -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.13.4/chess.min.js"></script>
    <script src="https://unpkg.com/chessground@9.1.0/chessground.js"></script>

    <script>
        // تهيئة اللعبة و الرقعة
        const chess = new Chess();
        let board = null;

        function renderBoard() {
            board = Chessground(document.getElementById('board'), {
                fen: chess.fen(),
                orientation: chess.turn() === 'w' ? 'white' : 'black',
                movable: {
                    free: false,
                    color: chess.turn(),
                    dests: computeLegalMoves(),
                    events: {
                        after: (orig, dest) => {
                            const move = chess.move({ from: orig, to: dest, promotion: 'q' });
                            if (move) {
                                updateUI();
                            }
                        }
                    }
                }
            });
        }

        // حساب النقلات القانونية للوحة
        function computeLegalMoves() {
            const dests = new Map();
            chess.SQUARES.forEach(s => {
                const ms = chess.moves({ square: s, verbose: true });
                if (ms.length) dests.set(s, ms.map(m => m.to));
            });
            return dests;
        }

        // تحديث الواجهة بعد كل نقلة
        function updateUI() {
            board.set({
                fen: chess.fen(),
                turnColor: chess.turn(),
                movable: {
                    color: chess.turn(),
                    dests: computeLegalMoves()
                }
            });
            document.getElementById('moves-list').innerText = chess.history().join(', ');
        }

        // دالة لتحميل ملف أو نص الـ PGN
        function loadPGN() {
            const pgnText = document.getElementById('pgn-input').value;
            if (chess.load_pgn(pgnText)) {
                updateUI();
                alert('تم تحميل المباراة بنجاح!');
            } else {
                alert('كود الـ PGN غير صحيح، تأكد من الصيغة.');
            }
        }

        // تشغيل الرقعة عند تحميل الصفحة
        window.onload = renderBoard;
    </script>
</body>
</html>
