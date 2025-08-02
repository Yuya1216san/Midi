<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
    <title>Web MIDI Kick Controller</title>
    <style>
        html, body {
            height: 100%;
            margin: 0;
            padding: 0;
            overflow: hidden;
            font-family: sans-serif;
            text-align: center;
        }
        #kick-button {
            width: 100%;
            height: 100%;
            background-color: #4CAF50;
            color: white;
            font-size: 2em;
            display: flex;
            justify-content: center;
            align-items: center;
            user-select: none; /* テキスト選択を無効化 */
            -webkit-user-select: none;
            touch-action: manipulation; /* タップの遅延をなくす */
        }
        #kick-button.active {
            background-color: #388E3C;
        }
    </style>
</head>
<body>
    <div id="kick-button">
        タップしてキックを鳴らす
    </div>
    <script>
        // Web MIDI APIが利用可能かチェック
        if (navigator.requestMIDIAccess) {
            navigator.requestMIDIAccess().then(onMIDISuccess, onMIDIFailure);
        } else {
            alert("お使いのブラウザはWeb MIDI APIをサポートしていません。Google Chromeなどをお試しください。");
        }

        let outputPort = null;
        let noteNumber = 36; // バスドラムの一般的なMIDIノートナンバー (C1)
        let velocity = 127; // 音の強さ (最大)
        const kickButton = document.getElementById('kick-button');

        function onMIDISuccess(midiAccess) {
            console.log("MIDI Access successful!");
            const outputs = midiAccess.outputs.values();
            
            // 最初のMIDI出力ポートを選択
            for (let output of outputs) {
                outputPort = output;
                console.log(`MIDI出力ポートが見つかりました: ${output.name}`);
                break;
            }

            if (!outputPort) {
                alert("MIDI出力ポートが見つかりませんでした。MIDIデバイスを接続しているか確認してください。");
                return;
            }

            // タッチイベントまたはマウスイベントを設定
            kickButton.addEventListener('touchstart', kickOn);
            kickButton.addEventListener('touchend', kickOff);
            kickButton.addEventListener('mousedown', kickOn);
            kickButton.addEventListener('mouseup', kickOff);
            kickButton.addEventListener('mouseleave', kickOff);
        }

        function onMIDIFailure(e) {
            console.error("MIDI Access failed:", e);
            alert("MIDIデバイスへのアクセスに失敗しました。");
        }

        function kickOn(event) {
            if (outputPort) {
                // ノートオンメッセージを送信
                outputPort.send([0x90, noteNumber, velocity]);
                kickButton.classList.add('active');
            }
            event.preventDefault(); // スクロールなどを防止
        }

        function kickOff(event) {
            if (outputPort) {
                // ノートオフメッセージを送信
                outputPort.send([0x80, noteNumber, 0]); // velocityは0でも可
                kickButton.classList.remove('active');
            }
            event.preventDefault(); // スクロールなどを防止
        }
    </script>
</body>
</html>
