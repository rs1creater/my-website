# my-website
import { useState, useEffect, useCallback } from "react";

const LEVELS = [
  { name: "Easy", ops: ["+", "-"], maxNum: 10, time: 30 },
  { name: "Medium", ops: ["+", "-", "×"], maxNum: 20, time: 25 },
  { name: "Hard", ops: ["+", "-", "×", "÷"], maxNum: 50, time: 20 },
];

function generateQuestion(level) {
  const cfg = LEVELS[level];
  const op = cfg.ops[Math.floor(Math.random() * cfg.ops.length)];
  let a, b, answer;

  if (op === "÷") {
    b = Math.floor(Math.random() * 9) + 2;
    answer = Math.floor(Math.random() * 10) + 1;
    a = b * answer;
  } else if (op === "×") {
    a = Math.floor(Math.random() * 12) + 1;
    b = Math.floor(Math.random() * 12) + 1;
    answer = a * b;
  } else if (op === "-") {
    a = Math.floor(Math.random() * cfg.maxNum) + 5;
    b = Math.floor(Math.random() * a);
    answer = a - b;
  } else {
    a = Math.floor(Math.random() * cfg.maxNum) + 1;
    b = Math.floor(Math.random() * cfg.maxNum) + 1;
    answer = a + b;
  }

  const wrong = new Set();
  while (wrong.size < 3) {
    const offset = Math.floor(Math.random() * 10) + 1;
    const w = Math.random() > 0.5 ? answer + offset : answer - offset;
    if (w !== answer && w > 0) wrong.add(w);
  }

  const options = [...wrong, answer].sort(() => Math.random() - 0.5);
  return { question: `${a} ${op} ${b}`, answer, options };
}

export default function MathGame() {
  const [screen, setScreen] = useState("home");
  const [level, setLevel] = useState(1);
  const [score, setScore] = useState(0);
  const [streak, setStreak] = useState(0);
  const [bestStreak, setBestStreak] = useState(0);
  const [q, setQ] = useState(null);
  const [timeLeft, setTimeLeft] = useState(0);
  const [feedback, setFeedback] = useState(null);
  const [totalAnswered, setTotalAnswered] = useState(0);
  const [correct, setCorrect] = useState(0);
  const [shake, setShake] = useState(false);
  const [pulse, setPulse] = useState(false);

  const nextQuestion = useCallback(() => {
    setQ(generateQuestion(level));
    setFeedback(null);
  }, [level]);

  useEffect(() => {
    if (screen === "playing") {
      setTimeLeft(LEVELS[level].time);
      nextQuestion();
    }
  }, [screen, level]);

  useEffect(() => {
    if (screen !== "playing") return;
    if (timeLeft <= 0) { setScreen("result"); return; }
    const t = setTimeout(() => setTimeLeft(t => t - 1), 1000);
    return () => clearTimeout(t);
  }, [timeLeft, screen]);

  const handleAnswer = (opt) => {
    if (feedback) return;
    const isCorrect = opt === q.answer;
    setFeedback(isCorrect ? "correct" : "wrong");
    setTotalAnswered(t => t + 1);

    if (isCorrect) {
      const bonus = streak >= 4 ? 3 : streak >= 2 ? 2 : 1;
      setScore(s => s + 10 * bonus);
      setCorrect(c => c + 1);
      setStreak(s => {
        const ns = s + 1;
        if (ns > bestStreak) setBestStreak(ns);
        return ns;
      });
      setPulse(true);
      setTimeout(() => setPulse(false), 400);
    } else {
      setStreak(0);
      setShake(true);
      setTimeout(() => setShake(false), 500);
    }

    setTimeout(() => nextQuestion(), 700);
  };

  const startGame = () => {
    setScore(0); setStreak(0); setTotalAnswered(0); setCorrect(0);
    setScreen("playing");
  };

  const timerPct = (timeLeft / LEVELS[level].time) * 100;
  const timerColor = timerPct > 50 ? "#00ff88" : timerPct > 25 ? "#ffcc00" : "#ff4444";

  // ... (baaki JSX aur styles upar se copy karein)
}
