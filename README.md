# -
簡易的なサイバーUIのタイマーです。10カウントダウンあり
import { useState, useEffect, useRef } from ‘react’;

export default function CyberTimer() {
const [totalSeconds, setTotalSeconds] = useState(300);
const [timeLeft, setTimeLeft] = useState(300);
const [isRunning, setIsRunning] = useState(false);
const [isFinished, setIsFinished] = useState(false);
const audioContextRef = useRef(null);

const initAudioContext = () => {
if (!audioContextRef.current) {
audioContextRef.current = new (window.AudioContext || window.webkitAudioContext)();
}
if (audioContextRef.current.state === ‘suspended’) {
audioContextRef.current.resume();
}
return audioContextRef.current;
};

const playCountdownBeep = () => {
const ctx = audioContextRef.current;
if (!ctx) return;

```
const oscillator = ctx.createOscillator();
const gainNode = ctx.createGain();

oscillator.connect(gainNode);
gainNode.connect(ctx.destination);

oscillator.frequency.setValueAtTime(880, ctx.currentTime);
oscillator.type = 'sine';

gainNode.gain.setValueAtTime(0.3, ctx.currentTime);
gainNode.gain.exponentialRampToValueAtTime(0.01, ctx.currentTime + 0.15);

oscillator.start(ctx.currentTime);
oscillator.stop(ctx.currentTime + 0.15);
```

};

const playAlarm = () => {
const ctx = audioContextRef.current;
if (!ctx) return;

```
for (let i = 0; i < 3; i++) {
  const oscillator = ctx.createOscillator();
  const gainNode = ctx.createGain();
  
  oscillator.connect(gainNode);
  gainNode.connect(ctx.destination);
  
  oscillator.type = 'square';
  const startTime = ctx.currentTime + i * 0.4;
  
  oscillator.frequency.setValueAtTime(523, startTime);
  oscillator.frequency.setValueAtTime(659, startTime + 0.1);
  oscillator.frequency.setValueAtTime(784, startTime + 0.2);
  
  gainNode.gain.setValueAtTime(0.2, startTime);
  gainNode.gain.exponentialRampToValueAtTime(0.01, startTime + 0.35);
  
  oscillator.start(startTime);
  oscillator.stop(startTime + 0.35);
}
```

};

useEffect(() => {
let interval;
if (isRunning && timeLeft > 0) {
interval = setInterval(() => {
setTimeLeft(prev => {
const newTime = prev - 1;
if (newTime <= 10 && newTime > 0) {
playCountdownBeep();
}
return newTime;
});
}, 1000);
} else if (timeLeft === 0 && isRunning) {
setIsRunning(false);
setIsFinished(true);
playAlarm();
}
return () => clearInterval(interval);
}, [isRunning, timeLeft]);

const formatTime = (seconds) => {
const mins = Math.floor(seconds / 60);
const secs = seconds % 60;
return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
};

const progress = totalSeconds > 0 ? (timeLeft / totalSeconds) * 100 : 0;

const handleStart = () => {
if (timeLeft > 0) {
initAudioContext();
setIsRunning(true);
setIsFinished(false);
}
};

const handlePause = () => setIsRunning(false);

const handleReset = () => {
setIsRunning(false);
setTimeLeft(totalSeconds);
setIsFinished(false);
};

const adjustTime = (delta) => {
if (!isRunning) {
const newTime = Math.max(0, Math.min(3600, totalSeconds + delta));
setTotalSeconds(newTime);
setTimeLeft(newTime);
setIsFinished(false);
}
};

return (
<div style={{
minHeight: ‘100vh’,
background: ‘linear-gradient(135deg, #0a0a0f 0%, #1a1a2e 50%, #0f0f1a 100%)’,
display: ‘flex’,
alignItems: ‘center’,
justifyContent: ‘center’,
fontFamily: ‘“Orbitron”, monospace’,
position: ‘relative’,
overflow: ‘hidden’
}}>
{/* Animated background grid */}
<div style={{
position: ‘absolute’,
inset: 0,
backgroundImage: `linear-gradient(rgba(0, 255, 255, 0.03) 1px, transparent 1px), linear-gradient(90deg, rgba(0, 255, 255, 0.03) 1px, transparent 1px)`,
backgroundSize: ‘50px 50px’,
animation: ‘gridMove 20s linear infinite’
}} />

```
  {/* Floating particles */}
  {[...Array(20)].map((_, i) => (
    <div key={i} style={{
      position: 'absolute',
      width: `${Math.random() * 4 + 2}px`,
      height: `${Math.random() * 4 + 2}px`,
      background: i % 2 === 0 ? '#00ffff' : '#ff00ff',
      borderRadius: '50%',
      left: `${Math.random() * 100}%`,
      top: `${Math.random() * 100}%`,
      opacity: 0.3,
      animation: `float ${Math.random() * 10 + 10}s ease-in-out infinite`,
      animationDelay: `${Math.random() * 5}s`,
      boxShadow: i % 2 === 0 ? '0 0 10px #00ffff' : '0 0 10px #ff00ff'
    }} />
  ))}

  {/* Main container */}
  <div style={{
    position: 'relative',
    padding: '60px',
    background: 'rgba(10, 10, 20, 0.8)',
    borderRadius: '30px',
    border: '1px solid rgba(0, 255, 255, 0.2)',
    boxShadow: `
      0 0 60px rgba(0, 255, 255, 0.1),
      inset 0 0 60px rgba(0, 255, 255, 0.05),
      0 0 120px rgba(255, 0, 255, 0.05)
    `,
    backdropFilter: 'blur(20px)'
  }}>
    {/* Corner decorations */}
    {['top-left', 'top-right', 'bottom-left', 'bottom-right'].map(pos => (
      <div key={pos} style={{
        position: 'absolute',
        width: '30px',
        height: '30px',
        border: '2px solid #00ffff',
        [pos.includes('top') ? 'top' : 'bottom']: '15px',
        [pos.includes('left') ? 'left' : 'right']: '15px',
        borderRadius: pos.includes('top') 
          ? (pos.includes('left') ? '10px 0 0 0' : '0 10px 0 0')
          : (pos.includes('left') ? '0 0 0 10px' : '0 0 10px 0'),
        borderBottom: pos.includes('top') ? 'none' : undefined,
        borderTop: pos.includes('bottom') ? 'none' : undefined,
        borderRight: pos.includes('left') ? 'none' : undefined,
        borderLeft: pos.includes('right') ? 'none' : undefined,
        opacity: 0.5
      }} />
    ))}

    {/* Title */}
    <div style={{
      textAlign: 'center',
      marginBottom: '40px',
      color: '#00ffff',
      fontSize: '14px',
      letterSpacing: '8px',
      textTransform: 'uppercase',
      textShadow: '0 0 20px rgba(0, 255, 255, 0.5)'
    }}>
      CYBER TIMER
    </div>

    {/* Timer display */}
    <div style={{
      position: 'relative',
      width: '280px',
      height: '280px',
      margin: '0 auto 40px'
    }}>
      {/* Outer ring */}
      <svg style={{
        position: 'absolute',
        width: '100%',
        height: '100%',
        transform: 'rotate(-90deg)'
      }}>
        <circle
          cx="140"
          cy="140"
          r="130"
          fill="none"
          stroke="rgba(0, 255, 255, 0.1)"
          strokeWidth="4"
        />
        <circle
          cx="140"
          cy="140"
          r="130"
          fill="none"
          stroke="url(#gradient)"
          strokeWidth="4"
          strokeLinecap="round"
          strokeDasharray={`${2 * Math.PI * 130}`}
          strokeDashoffset={`${2 * Math.PI * 130 * (1 - progress / 100)}`}
          style={{
            transition: 'stroke-dashoffset 0.5s ease',
            filter: 'drop-shadow(0 0 10px #00ffff)'
          }}
        />
        <defs>
          <linearGradient id="gradient" x1="0%" y1="0%" x2="100%" y2="100%">
            <stop offset="0%" stopColor="#00ffff" />
            <stop offset="50%" stopColor="#ff00ff" />
            <stop offset="100%" stopColor="#00ffff" />
          </linearGradient>
        </defs>
      </svg>

      {/* Inner glow ring */}
      <div style={{
        position: 'absolute',
        inset: '20px',
        borderRadius: '50%',
        background: 'radial-gradient(circle, rgba(0, 255, 255, 0.05) 0%, transparent 70%)',
        border: '1px solid rgba(0, 255, 255, 0.1)'
      }} />

      {/* Time display */}
      <div style={{
        position: 'absolute',
        inset: 0,
        display: 'flex',
        flexDirection: 'column',
        alignItems: 'center',
        justifyContent: 'center'
      }}>
        <div style={{
          fontSize: '64px',
          fontWeight: '700',
          color: isFinished ? '#ff00ff' : '#ffffff',
          textShadow: isFinished 
            ? '0 0 30px #ff00ff, 0 0 60px #ff00ff'
            : '0 0 20px rgba(0, 255, 255, 0.5)',
          letterSpacing: '4px',
          animation: isFinished ? 'pulse 0.5s ease-in-out infinite' : 'none'
        }}>
          {formatTime(timeLeft)}
        </div>
        <div style={{
          fontSize: '12px',
          color: 'rgba(0, 255, 255, 0.6)',
          letterSpacing: '4px',
          marginTop: '10px'
        }}>
          {isRunning ? 'RUNNING' : isFinished ? 'COMPLETE' : 'READY'}
        </div>
      </div>
    </div>

    {/* Time adjustment */}
    <div style={{
      display: 'flex',
      justifyContent: 'center',
      gap: '20px',
      marginBottom: '30px'
    }}>
      {[
        { label: '-1m', delta: -60 },
        { label: '-10s', delta: -10 },
        { label: '+10s', delta: 10 },
        { label: '+1m', delta: 60 }
      ].map(({ label, delta }) => (
        <button
          key={label}
          onClick={() => adjustTime(delta)}
          disabled={isRunning}
          style={{
            padding: '8px 16px',
            background: 'transparent',
            border: '1px solid rgba(0, 255, 255, 0.3)',
            borderRadius: '8px',
            color: isRunning ? 'rgba(0, 255, 255, 0.3)' : '#00ffff',
            fontSize: '12px',
            fontFamily: '"Orbitron", monospace',
            cursor: isRunning ? 'not-allowed' : 'pointer',
            transition: 'all 0.3s ease',
            letterSpacing: '2px'
          }}
          onMouseEnter={e => {
            if (!isRunning) {
              e.target.style.background = 'rgba(0, 255, 255, 0.1)';
              e.target.style.boxShadow = '0 0 20px rgba(0, 255, 255, 0.3)';
            }
          }}
          onMouseLeave={e => {
            e.target.style.background = 'transparent';
            e.target.style.boxShadow = 'none';
          }}
        >
          {label}
        </button>
      ))}
    </div>

    {/* Control buttons */}
    <div style={{
      display: 'flex',
      justifyContent: 'center',
      gap: '20px'
    }}>
      {!isRunning ? (
        <button
          onClick={handleStart}
          style={{
            padding: '16px 48px',
            background: 'linear-gradient(135deg, rgba(0, 255, 255, 0.2), rgba(255, 0, 255, 0.2))',
            border: '2px solid #00ffff',
            borderRadius: '12px',
            color: '#00ffff',
            fontSize: '16px',
            fontFamily: '"Orbitron", monospace',
            cursor: 'pointer',
            letterSpacing: '4px',
            transition: 'all 0.3s ease',
            boxShadow: '0 0 30px rgba(0, 255, 255, 0.2)'
          }}
          onMouseEnter={e => {
            e.target.style.background = 'linear-gradient(135deg, rgba(0, 255, 255, 0.4), rgba(255, 0, 255, 0.4))';
            e.target.style.boxShadow = '0 0 50px rgba(0, 255, 255, 0.4)';
            e.target.style.transform = 'scale(1.05)';
          }}
          onMouseLeave={e => {
            e.target.style.background = 'linear-gradient(135deg, rgba(0, 255, 255, 0.2), rgba(255, 0, 255, 0.2))';
            e.target.style.boxShadow = '0 0 30px rgba(0, 255, 255, 0.2)';
            e.target.style.transform = 'scale(1)';
          }}
        >
          START
        </button>
      ) : (
        <button
          onClick={handlePause}
          style={{
            padding: '16px 48px',
            background: 'linear-gradient(135deg, rgba(255, 0, 255, 0.2), rgba(255, 100, 100, 0.2))',
            border: '2px solid #ff00ff',
            borderRadius: '12px',
            color: '#ff00ff',
            fontSize: '16px',
            fontFamily: '"Orbitron", monospace',
            cursor: 'pointer',
            letterSpacing: '4px',
            transition: 'all 0.3s ease',
            boxShadow: '0 0 30px rgba(255, 0, 255, 0.2)'
          }}
          onMouseEnter={e => {
            e.target.style.boxShadow = '0 0 50px rgba(255, 0, 255, 0.4)';
            e.target.style.transform = 'scale(1.05)';
          }}
          onMouseLeave={e => {
            e.target.style.boxShadow = '0 0 30px rgba(255, 0, 255, 0.2)';
            e.target.style.transform = 'scale(1)';
          }}
        >
          PAUSE
        </button>
      )}
      <button
        onClick={handleReset}
        style={{
          padding: '16px 32px',
          background: 'transparent',
          border: '2px solid rgba(255, 255, 255, 0.3)',
          borderRadius: '12px',
          color: 'rgba(255, 255, 255, 0.6)',
          fontSize: '16px',
          fontFamily: '"Orbitron", monospace',
          cursor: 'pointer',
          letterSpacing: '4px',
          transition: 'all 0.3s ease'
        }}
        onMouseEnter={e => {
          e.target.style.borderColor = '#ffffff';
          e.target.style.color = '#ffffff';
          e.target.style.transform = 'scale(1.05)';
        }}
        onMouseLeave={e => {
          e.target.style.borderColor = 'rgba(255, 255, 255, 0.3)';
          e.target.style.color = 'rgba(255, 255, 255, 0.6)';
          e.target.style.transform = 'scale(1)';
        }}
      >
        RESET
      </button>
    </div>

    {/* Status bar */}
    <div style={{
      marginTop: '40px',
      display: 'flex',
      justifyContent: 'center',
      gap: '30px',
      fontSize: '10px',
      color: 'rgba(0, 255, 255, 0.4)',
      letterSpacing: '2px'
    }}>
      <span>SYS:ONLINE</span>
      <span>▪</span>
      <span>MODE:COUNTDOWN</span>
      <span>▪</span>
      <span>PWR:100%</span>
    </div>
  </div>

  <style>{`
    @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&display=swap');
    
    @keyframes gridMove {
      0% { transform: translate(0, 0); }
      100% { transform: translate(50px, 50px); }
    }
    
    @keyframes float {
      0%, 100% { transform: translateY(0) translateX(0); opacity: 0.3; }
      50% { transform: translateY(-30px) translateX(20px); opacity: 0.6; }
    }
    
    @keyframes pulse {
      0%, 100% { opacity: 1; }
      50% { opacity: 0.5; }
    }
  `}</style>
</div>
```

);
}
