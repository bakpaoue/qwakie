# qwakie
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Alarm Clock</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;600&display=swap');

  body {
    font-family: 'Poppins', sans-serif;
    background: linear-gradient(135deg, #eaebd0, #ecec9b);
    color: #fff;
    margin: 0;
    min-height: 100vh;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    padding: 20px;
  }

  h1 {
    font-weight: 600;
    font-size: 3rem;
    margin-bottom: 0.2em;
    text-shadow: 0 0 15px rgba(119, 81, 10, 0.493);
  }

  #clock {
    font-size: 4rem;
    margin-bottom: 40px;
    letter-spacing: 6px;
    text-shadow: 0 0 20px rgba(116, 112, 94, 0.25);
  }

  .alarm-controls {
    background: #e5d0ac;
    padding: 25px 35px;
    border-radius: 15px;
    box-shadow: 0 8px 32px rgba(90, 90, 90, 0.15);
    display: flex;
    flex-direction: column;
    align-items: center;
    max-width: 350px;
    width: 100%;
    gap: 20px;
  }

  .label-main {
    font-size: 1.1rem;
    font-weight: 300;
    user-select: none;
  }
  .label-sub {
    font-size: 0.9rem;
    font-weight: 300;
    color: #555;
    margin-top: -10px;
    user-select: none;
  }

  .inputs-row {
    display: flex;
    gap: 15px;
    width: 100%;
    justify-content: center;
  }

  input[type="number"] {
    width: 60px;
    padding: 8px 10px;
    font-size: 1.3rem;
    border-radius: 8px;
    border: none;
    text-align: center;
    font-weight: 600;
    box-shadow: inset 0 0 10px rgba(255,255,255,0.3);
    color: #222;
    transition: box-shadow 0.3s ease;
  }

  input[type="number"]:focus {
    outline: none;
    box-shadow: 0 0 12px #ca3e5c;
  }

  button {
    cursor: pointer;
    background: #da6c6c;
    border: none;
    padding: 12px 25px;
    font-size: 1.2rem;
    border-radius: 30px;
    font-weight: 600;
    color: white;
    box-shadow: 0 4px 15px rgba(134, 53, 53, 0.7);
    transition: background-color 0.3s ease;
    flex-grow: 1;
    min-width: 120px;
    user-select: none;
  }
  button:hover:not(:disabled) {
    background: #cd5656;
  }
  button:disabled {
    background: #ccc;
    cursor: not-allowed;
    box-shadow: none;
  }

  #notification {
    margin-top: 30px;
    font-weight: 700;
    font-size: 1.4rem;
    color: #ffffff;
    text-shadow: 0 0 10px #332d56;
    min-height: 24px;
  }
</style>
</head>
<body>
  <h1>Alarm Qwack</h1>
  <div id="clock" aria-live="polite" aria-atomic="true">00:00:00</div>

  <form class="alarm-controls" onsubmit="event.preventDefault(); setAlarm();">
    <div class="label-main">Set Alarm Time here!</div>
    <div class="label-sub">(24h format)</div>
    <div class="inputs-row">
      <input type="number" id="hourInput" min="0" max="23" placeholder="HH" required aria-label="Hour" />
      <input type="number" id="minuteInput" min="0" max="59" placeholder="MM" required aria-label="Minute" />
      <input type="number" id="secondInput" min="0" max="59" placeholder="SS" required aria-label="Second" />
    </div>
    <button type="submit" id="setAlarmBtn">Set Alarm</button>
    <button type="button" id="stopAlarmBtn" style="display:none;">Stop Alarm</button>
  </form>

  <div id="notification" role="alert" aria-live="assertive"></div>

<script>
  const clockElem = document.getElementById('clock');
  const hourInput = document.getElementById('hourInput');
  const minuteInput = document.getElementById('minuteInput');
  const secondInput = document.getElementById('secondInput');
  const setAlarmBtn = document.getElementById('setAlarmBtn');
  const stopAlarmBtn = document.getElementById('stopAlarmBtn');
  const notification = document.getElementById('notification');

  let alarmTime = null;
  let alarmTimeout = null;
  let isAlarmRinging = false;
  let audioCtx;

  function updateClock() {
    const now = new Date();
    const hh = now.getHours().toString().padStart(2, '0');
    const mm = now.getMinutes().toString().padStart(2, '0');
    const ss = now.getSeconds().toString().padStart(2, '0');
    clockElem.textContent = `${hh}:${mm}:${ss}`;

    if (alarmTime === `${hh}:${mm}:${ss}` && !isAlarmRinging) {
      startAlarm();
    }
  }

  function setAlarm() {
    const hh = hourInput.value.padStart(2, '0');
    const mm = minuteInput.value.padStart(2, '0');
    const ss = secondInput.value.padStart(2, '0');

    if (
      isNaN(hh) || hh < 0 || hh > 23 ||
      isNaN(mm) || mm < 0 || mm > 59 ||
      isNaN(ss) || ss < 0 || ss > 59
    ) {
      alert('Silakan masukkan nilai waktu yang valid.');
      return;
    }

    alarmTime = `${hh}:${mm}:${ss}`;
    notification.textContent = `Alarm disetel untuk ${alarmTime}`;
    setAlarmBtn.disabled = true;
    hourInput.disabled = true;
    minuteInput.disabled = true;
    secondInput.disabled = true;
  }

  function startAlarm() {
    isAlarmRinging = true;
    notification.textContent = "⏰ Alarm mu berbunyi! Klik 'Stop Alarm' untuk menghentikannya.";
    stopAlarmBtn.style.display = 'inline-block';
    setAlarmBtn.disabled = true;

    if (!audioCtx) {
      audioCtx = new (window.AudioContext || window.webkitAudioContext)();
    }

    playBeep();
  }

  let oscillator, gainNode;
  function playBeep() {
    if (!isAlarmRinging) return;

    oscillator = audioCtx.createOscillator();
    gainNode = audioCtx.createGain();

    oscillator.type = 'sine';
    oscillator.frequency.setValueAtTime(1000, audioCtx.currentTime);
    gainNode.gain.setValueAtTime(0.1, audioCtx.currentTime);

    oscillator.connect(gainNode);
    gainNode.connect(audioCtx.destination);

    oscillator.start();

    let on = true;
    let beepInterval = setInterval(() => {
      if (!isAlarmRinging) {
        clearInterval(beepInterval);
        oscillator.stop();
        return;
      }
      gainNode.gain.setValueAtTime(on ? 0.1 : 0, audioCtx.currentTime);
      on = !on;
    }, 500);
  }

  function stopAlarm() {
    isAlarmRinging = false;
    alarmTime = null;
    notification.textContent = 'Alarm dihentikan.';
    stopAlarmBtn.style.display = 'none';
    setAlarmBtn.disabled = false;
    hourInput.disabled = false;
    minuteInput.disabled = false;
    secondInput.disabled = false;

    if (oscillator) {
      oscillator.stop();
      oscillator.disconnect();
    }
    if (gainNode) gainNode.disconnect();
  }

  setAlarmBtn.addEventListener('click', setAlarm);
  stopAlarmBtn.addEventListener('click', stopAlarm);

  setInterval(updateClock, 1000);
  updateClock();
</script>

</body>
</html>

