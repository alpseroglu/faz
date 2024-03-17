<!DOCTYPE html>
<html lang="tr">

<head>
    <meta charset="UTF-8">
    <title>Canvas Resim Animasyonu</title>
    <style>
        body {
            margin: 0;
            height: 100vh;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding-top: 10px;
        }

        #myCanvas {
            border: 1px solid #000;
            margin-top: 10px;
        }

        table {
            margin-bottom: 10px;
        }
    </style>
</head>

<body>
    <table>
        <tr>
            <td><input type="file" id="imageLoader" name="imageLoader" /></td>
            <td><label for="totalDuration">Toplam Süre (saniye):</label></td>
            <td><input type="number" id="totalDuration" min="1" value="10"></td>
        </tr>
        <tr>
            <td><label for="phase1Start">Faz1 Başlangıç:</label></td>
            <td><input type="number" id="phase1Start" min="0" value="0"></td>
            <td><label for="phase1End">Faz1 Bitiş:</label></td>
            <td><input type="number" id="phase1End" min="0" value="5"></td>
            <td>
                <select id="phase1Animation">
                    <option value="movement">Hareket</option>
                    <option value="scale">Ölçeklendirme</option>
                    <option value="rotation">Dönme</option>
                    <option value="blink">Yanıp Sönme</option>
                    <option value="colorChange">Renk Değiştirme</option>
                </select>
            </td>
        </tr>
        <tr>
            <td><label for="phase2Start">Faz2 Başlangıç:</label></td>
            <td><input type="number" id="phase2Start" min="0" value="5"></td>
            <td><label for="phase2End">Faz2 Bitiş:</label></td>
            <td><input type="number" id="phase2End" min="0" value="10"></td>
            <td>
                <select id="phase2Animation">
                    <option value="movement">Hareket</option>
                    <option value="scale">Ölçeklendirme</option>
                    <option value="rotation">Dönme</option>
                    <option value="blink">Yanıp Sönme</option>
                    <option value="colorChange">Renk Değiştirme</option>
                </select>
            </td>
        </tr>
    </table>
    <button onclick="startRecording()">Kaydı Başlat</button>
    <canvas id="myCanvas" width="500" height="500"></canvas>
    <button onclick="downloadVideo()" style="display:none;">İndir</button>

    <script>
        const canvas = document.getElementById('myCanvas');
        const ctx = canvas.getContext('2d');
        let mediaRecorder;
        let recordedChunks = [];
        let image = new Image();
        let animationFrameId;
        let x = 0, y = 50;
        let scale = 1;
        let direction = 1;
        let angle = 0;
        let colorIndex = 0;
        let colors = ['red', 'green', 'blue', 'yellow', 'purple'];
        let totalDuration = document.getElementById('totalDuration');
        let phase1Start = document.getElementById('phase1Start');
        let phase1End = document.getElementById('phase1End');
        let phase1Animation = document.getElementById('phase1Animation');
        let phase2Start = document.getElementById('phase2Start');
        let phase2End = document.getElementById('phase2End');
        let phase2Animation = document.getElementById('phase2Animation');
        let downloadButton = document.querySelector('button[onclick="downloadVideo()"]');

        document.getElementById('imageLoader').addEventListener('change', loadImage, false);

        function loadImage(e) {
            const reader = new FileReader();
            reader.onload = function (event) {
                image.onload = function () {
                    drawImage();
                }
                image.src = event.target.result;
            }
            reader.readAsDataURL(e.target.files[0]);
        }

        function drawImage() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(image, x, y, 100, 100);
        }

        function startRecording() {
            recordedChunks = [];
            const stream = canvas.captureStream(30);
            mediaRecorder = new MediaRecorder(stream);
            mediaRecorder.ondataavailable = function (e) {
                if (e.data.size > 0) {
                    recordedChunks.push(e.data);
                }
            };
            mediaRecorder.onstop = function () {
                downloadButton.style.display = 'block';
            };
            mediaRecorder.start();

            const duration = parseInt(totalDuration.value, 10) * 1000;
            const phase1StartTime = parseInt(phase1Start.value, 10) * 1000;
            const phase1EndTime = parseInt(phase1End.value, 10) * 1000;
            const phase2StartTime = parseInt(phase2Start.value, 10) * 1000;
            const phase2EndTime = parseInt(phase2End.value, 10) * 1000;

            setTimeout(() => {
                switch (phase1Animation.value) {
                    case "movement":
                        animateMovement();
                        break;
                    case "scale":
                        animateScale();
                        break;
                    case "rotation":
                        animateRotation();
                        break;
                    case "blink":
                        animateBlink();
                        break;
                    case "colorChange":
                        animateColorChange();
                        break;
                }
            }, phase1StartTime);

            setTimeout(() => { cancelAnimationFrame(animationFrameId); }, phase1EndTime);

            setTimeout(() => {
                switch (phase2Animation.value) {
                    case "movement":
                        animateMovement();
                        break;
                    case "scale":
                        animateScale();
                        break;
                    case "rotation":
                        animateRotation();
                        break;
                    case "blink":
                        animateBlink();
                        break;
                    case "colorChange":
                        animateColorChange();
                        break;
                }
            }, phase2StartTime);

            setTimeout(() => { cancelAnimationFrame(animationFrameId); }, phase2EndTime);

            setTimeout(() => {
                mediaRecorder.stop();
                cancelAnimationFrame(animationFrameId);
            }, duration);
        }

        function animateMovement() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(image, x, y, 100, 100);
            x += 2;
            if (x > canvas.width) {
                x = -100;
            }
            animationFrameId = requestAnimationFrame(animateMovement);
        }

        function animateScale() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.drawImage(image, x, y, 100 * scale, 100 * scale);
            if (scale >= 2 || scale <= 0.5) {
                direction *= -1;
            }
            scale += direction * 0.01;
            animationFrameId = requestAnimationFrame(animateScale);
        }

        function animateRotation() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.save();
            ctx.translate(x + 50, y + 50);
            ctx.rotate(angle * Math.PI / 180);
            ctx.drawImage(image, -50, -50, 100, 100);
            ctx.restore();
            angle += 2;
            animationFrameId = requestAnimationFrame(animateRotation);
        }

        function animateBlink() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            if (Math.floor(Date.now() / 250) % 2 === 0) {
                ctx.drawImage(image, x, y, 100, 100);
            }
            animationFrameId = requestAnimationFrame(animateBlink);
        }

        function animateColorChange() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = colors[colorIndex];
            ctx.fillRect(x, y, 100, 100);
            colorIndex = (colorIndex + 1) % colors.length;
            animationFrameId = requestAnimationFrame(animateColorChange);
        }

        function downloadVideo() {
            const blob = new Blob(recordedChunks, {
                type: 'video/webm'
            });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.style.display = 'none';
            a.href = url;
            a.download = 'canvas-animation.webm';
            document.body.appendChild(a);
            a.click();
            window.URL.revokeObjectURL(url);
        }
    </script>
</body>

</html>
