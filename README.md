<!DOCTYPE html>
<html>
<head>
  <title>AI Interviewer</title>

  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f6f8;
      display: flex;
      justify-content: center;
      align-items: center;
      height: 100vh;
    }

    .box {
      background: white;
      width: 650px;
      padding: 25px;
      border-radius: 10px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.1);
      text-align: center;
    }

    select, button {
      width: 100%;
      padding: 10px;
      margin-top: 6px;
      border-radius: 6px;
      font-weight: bold;
      border: none;
    }

    button { cursor: pointer; }

    .start { background: #4f46e5; color: white; }
    .answer { background: #10b981; color: white; }
    .next { background: #f59e0b; color: white; }
    .switch { background: #6366f1; color: white; }
    .end { background: #ef4444; color: white; }

    #question {
      background: #eef2ff;
      padding: 12px;
      border-radius: 6px;
      margin-top: 12px;
      font-weight: bold;
    }

    .progress-container {
      width: 100%;
      background: #e5e7eb;
      height: 10px;
      border-radius: 5px;
      margin-top: 10px;
    }

    .progress-bar {
      height: 10px;
      background: #4f46e5;
      width: 0%;
      border-radius: 5px;
    }

    pre {
      background: #f9fafb;
      padding: 10px;
      margin-top: 12px;
      border-radius: 6px;
      text-align: left;
      white-space: pre-wrap;
      min-height: 200px;
    }
  </style>
</head>

<body>

<div class="box">
  <h2>🎤 AI Interviewer</h2>

  <select id="category">
    <option value="">-- Select Interview Category --</option>
    <option value="technical">Technical</option>
    <option value="coding">Coding</option>
    <option value="project">Project</option>
    <option value="hr">HR</option>
  </select>

  <div class="progress-container">
    <div class="progress-bar" id="progressBar"></div>
  </div>

  <div id="question">Select category & start interview</div>

  <button class="start" onclick="startInterview()">Start Interview</button>
  <button class="answer" onclick="startListening()">🎙 Answer</button>
  <button class="next" onclick="nextQuestion()">Next Question</button>
  <button class="switch" onclick="switchCategory()">Switch Category</button>
  <button class="end" onclick="endInterview()">End Interview</button>

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
  if (!category) return alert("Select category");

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

  speak("Category switched. Starting new questions.");
  askQuestion();
}

/* ================= AI-LIKE EVALUATION (ELABORATED) ================= */
function evaluateAnswer(answer) {
  const text = answer.toLowerCase();
  const words = text.split(" ").length;

  let score = 0;
  let review = [];

  // Depth
  if (words < 12) {
    score += 2;
    review.push("Your answer is quite brief and lacks sufficient depth.");
  } else if (words < 30) {
    score += 4;
    review.push("You covered the basics, but the explanation could be deeper.");
  } else {
    score += 6;
    review.push("Your answer is well-detailed and shows good understanding.");
  }

  // Explanation
  if (text.includes("because") || text.includes("for example") || text.includes("such as")) {
    score += 2;
    review.push("Good use of reasoning or examples to support your explanation.");
  } else {
    review.push("Try adding examples or reasoning to strengthen your answer.");
  }

  // Technical terms
  const keywords = ["process","thread","algorithm","complexity","api","database","server"];
  const hits = keywords.filter(k => text.includes(k)).length;

  if (hits >= 2) {
    score += 2;
    review.push("You used relevant technical terminology appropriately.");
  } else {
    review.push("Including more technical terms would improve clarity.");
  }

  // Confidence
  if (text.includes("maybe") || text.includes("i think")) {
    score -= 1;
    review.push("Try to sound more confident while answering.");
  } else {
    review.push("Your tone sounds confident and professional.");
  }

  score = Math.max(1, Math.min(10, score));

  return {
    score,
    feedback: review.join("\n")
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

    if (finalText && event.results[event.results.length - 1].isFinal) {
      recognition.stop();

      if (!introDone) {
        introDone = true;
        speak("That's great. Let's move forward with questions.");
        questions = questionBank[category];
        index = 0;
        askQuestion();
        return;
      }

      const evaluation = evaluateAnswer(finalText);

      interviewData.push({
        question: questions[index],
        answer: finalText,
        score: evaluation.score
      });

      document.getElementById("answer").innerText +=
        `\n\n🧠 AI Score: ${evaluation.score}/10\n\n📌 AI Review:\n${evaluation.feedback}`;
    }
  };
}

/* ================= FINAL REPORT ================= */
function endInterview() {
  interviewActive = false;

  let report = "📄 FINAL INTERVIEW REPORT\n\n";
  let total = 0;

  interviewData.forEach((item, i) => {
    report += `Q${i+1}: ${item.question}\n`;
    report += `Score: ${item.score}/10\n\n`;
    total += item.score;
  });

  const avg = (total / (interviewData.length || 1)).toFixed(1);

  report += `⭐ Average Score: ${avg}/10\n`;
  report += "💡 Overall Tip: Use structured answers with examples and confidence.";

  document.getElementById("question").innerText = "Interview Ended ✅";
  document.getElementById("answer").innerText = report;
  speak("Interview ended. Final report generated.");
}
</script>

</body>
</html>
