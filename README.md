<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>AI Interviewer</title>

  <style>
    * { box-sizing: border-box; }

    body {
      margin: 0;
      font-family: "Segoe UI", sans-serif;
      background: linear-gradient(135deg, #6366f1, #22c55e);
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    .box {
      background: rgba(255,255,255,0.15);
      backdrop-filter: blur(15px);
      width: 760px;
      padding: 30px;
      border-radius: 18px;
      box-shadow: 0 20px 40px rgba(0,0,0,0.25);
      text-align: center;
      color: white;
      animation: fadeIn 0.8s ease;
    }

    h2 { margin-top: 0; }

    select, button {
      width: 100%;
      padding: 12px;
      margin-top: 10px;
      border-radius: 10px;
      font-weight: bold;
      border: none;
      cursor: pointer;
      transition: all 0.25s ease;
    }

    select {
      background: rgba(255,255,255,0.95);
      color: #111;
    }

    button:hover {
      transform: translateY(-2px);
      box-shadow: 0 8px 15px rgba(0,0,0,0.3);
    }

    .start { background: #4f46e5; }
    .answer { background: #10b981; }
    .next { background: #f59e0b; }
    .switch { background: #6366f1; }
    .end { background: #ef4444; }

    #question {
      background: rgba(255,255,255,0.25);
      padding: 16px;
      border-radius: 12px;
      margin-top: 15px;
      min-height: 60px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: 600;
    }

    .progress-container {
      width: 100%;
      background: rgba(255,255,255,0.3);
      height: 10px;
      border-radius: 10px;
      margin-top: 15px;
      overflow: hidden;
    }

    .progress-bar {
      height: 10px;
      background: linear-gradient(90deg, #22c55e, #4f46e5);
      width: 0%;
      transition: width 0.4s ease;
    }

    pre {
      background: rgba(0,0,0,0.35);
      padding: 15px;
      margin-top: 15px;
      border-radius: 12px;
      text-align: left;
      min-height: 220px;
      white-space: pre-wrap;
      font-size: 14px;
    }

    @keyframes fadeIn {
      from { opacity: 0; transform: translateY(20px); }
      to { opacity: 1; transform: translateY(0); }
    }
  </style>
</head>

<body>

<div class="box">
  <h2>🤖 AI Interviewer</h2>

  <select id="category">
    <option value="">Select Interview Category</option>
    <option value="technical">Technical</option>
    <option value="coding">Coding</option>
    <option value="project">Project</option>
    <option value="hr">HR</option>
  </select>

  <div class="progress-container">
    <div class="progress-bar" id="progressBar"></div>
  </div>

  <div id="question">Select category & start interview</div>

  <button class="start" onclick="startInterview()">🚀 Start Interview</button>
  <button class="answer" onclick="startListening()">🎙 Answer</button>
  <button class="next" onclick="nextQuestion()">➡ Next Question</button>
  <button class="switch" onclick="switchCategory()">🔄 Switch Category</button>
  <button class="end" onclick="endInterview()">⛔ End Interview</button>

  <pre id="answer"></pre>
</div>

<script>
/* ================= QUESTIONS ================= */
const introQuestion = "Introduce yourself";

const questionBank = {
  technical: [
    "What is time complexity?",
    "Difference between process and thread?",
    "What is deadlock?",
    "Difference between TCP and UDP?",
    "What is REST API?"
  ],
  coding: [
    "Explain binary search.",
    "What is two sum problem?",
    "Reverse an array.",
    "Find missing number from 1 to N."
  ],
  project: [
    "Explain your project.",
    "Challenges you faced?",
    "Tech stack used?",
    "How would you scale it?"
  ],
  hr: [
    "Why should we hire you?",
    "Your strengths?",
    "Your weaknesses?",
    "Where do you see yourself in 5 years?"
  ]
};

/* ================= STATE ================= */
let questions = [];
let index = 0;
let category = "";
let interviewActive = false;
let introDone = false;
let interviewData = [];

/* ================= UTILS ================= */
function speak(text) {
  speechSynthesis.speak(new SpeechSynthesisUtterance(text));
}

function updateProgress() {
  const percent = Math.round((index / questions.length) * 100);
  document.getElementById("progressBar").style.width = percent + "%";
}

/* ================= FLOW ================= */
function startInterview() {
  category = document.getElementById("category").value;
  if (!category) return alert("Select a category");

  questions = [introQuestion];
  index = 0;
  introDone = false;
  interviewActive = true;
  interviewData = [];

  askQuestion();
}

function askQuestion() {
  if (index >= questions.length) {
    document.getElementById("question").innerText = "Interview Completed ✅";
    speak("Interview completed.");
    return;
  }

  document.getElementById("question").innerText =
    `Question ${index + 1}: ${questions[index]}`;

  speak(questions[index]);
  updateProgress();
}

function nextQuestion() {
  if (!interviewActive) return;
  index++;
  askQuestion();
}

function switchCategory() {
  category = document.getElementById("category").value;
  if (!category) return;

  questions = questionBank[category];
  index = 0;
  introDone = true;
  speak("Category switched.");
  askQuestion();
}

/* ================= AI REVIEW SYSTEM ================= */
function evaluateAnswer(answer) {
  const text = answer.toLowerCase();
  const words = text.trim().split(/\s+/).length;

  let score = 0;
  let review = [];

  if (words < 12) {
    score += 2;
    review.push("❌ Answer is very brief and lacks depth.");
  } else if (words < 30) {
    score += 4;
    review.push("⚠️ Covers basics but needs deeper explanation.");
  } else {
    score += 6;
    review.push("✅ Well-detailed explanation.");
  }

  if (text.includes("because") || text.includes("for example") || text.includes("such as")) {
    score += 2;
    review.push("✅ Good use of reasoning/examples.");
  } else {
    review.push("⚠️ Add examples or reasoning.");
  }

  const keywords = ["process","thread","algorithm","complexity","api","database","server"];
  if (keywords.filter(k => text.includes(k)).length >= 2) {
    score += 2;
    review.push("✅ Used relevant technical terms.");
  } else {
    review.push("⚠️ Use more technical terminology.");
  }

  if (text.includes("maybe") || text.includes("i think")) {
    score -= 1;
    review.push("❌ Sounds unsure. Be confident.");
  } else {
    review.push("✅ Confident tone.");
  }

  score = Math.max(1, Math.min(10, score));
  const stars = "⭐".repeat(Math.round(score / 2));

  return {
    score,
    feedback: `📊 AI ANSWER REVIEW\n--------------------\n${review.join("\n")}\n\n⭐ Rating: ${stars}\n🎯 Final Score: ${score}/10`
  };
}

/* ================= SPEECH ================= */
function startListening() {
  if (!interviewActive) return;

  const recognition = new webkitSpeechRecognition();
  recognition.lang = "en-US";
  recognition.interimResults = true;

  let finalText = "";
  document.getElementById("answer").innerText = "🎙 Listening...";

  recognition.start();

  recognition.onresult = (event) => {
    let interim = "";

    for (let i = event.resultIndex; i < event.results.length; i++) {
      const t = event.results[i][0].transcript;
      event.results[i].isFinal ? finalText += t + " " : interim += t;
    }

    document.getElementById("answer").innerText =
      "Live:\n" + interim + "\n\nFinal:\n" + finalText;

    if (event.results[event.results.length - 1].isFinal) {
      recognition.stop();

      if (!introDone) {
        introDone = true;
        questions = questionBank[category];
        index = 0;
        speak("Let's begin the interview.");
        askQuestion();
        return;
      }

      const evalResult = evaluateAnswer(finalText);

      interviewData.push({
        question: questions[index],
        score: evalResult.score
      });

      document.getElementById("answer").innerText +=
        `\n\n🧠 AI Score: ${evalResult.score}/10\n\n${evalResult.feedback}`;
    }
  };
}

/* ================= FINAL REPORT ================= */
function endInterview() {
  interviewActive = false;

  let report = "📄 FINAL INTERVIEW REPORT\n\n";
  let total = 0;

  interviewData.forEach((q, i) => {
    report += `Q${i+1}: ${q.question}\nScore: ${q.score}/10\n\n`;
    total += q.score;
  });

  report += `⭐ Average Score: ${(total / (interviewData.length || 1)).toFixed(1)}/10\n`;
  report += "💡 Tip: Structure answers with definition → example → conclusion.";

  document.getElementById("question").innerText = "Interview Ended ✅";
  document.getElementById("answer").innerText = report;
  speak("Interview ended. Final report generated.");
}
</script>

</body>
</html>
