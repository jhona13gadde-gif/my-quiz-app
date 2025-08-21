<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Quiz Generator</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap');
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
        }
        .container {
            max-width: 900px;
            margin: auto;
            padding: 2rem;
            background-color: white;
            border-radius: 1.5rem;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
        }
        .btn-primary {
            background-color: #2563eb;
            color: white;
            padding: 0.75rem 1.5rem;
            border-radius: 0.75rem;
            font-weight: 600;
            transition: background-color 0.2s;
        }
        .btn-primary:hover {
            background-color: #1d4ed8;
        }
        .input-field {
            border: 2px solid #e5e7eb;
            padding: 0.5rem 1rem;
            border-radius: 0.75rem;
            width: 100%;
            transition: border-color 0.2s;
        }
        .input-field:focus {
            outline: none;
            border-color: #2563eb;
        }
        .timer-display {
            font-size: 1.5rem;
            font-weight: 700;
            color: #1f2937;
        }
        .timer-buttons button {
            background-color: #f3f4f6;
            color: #1f2937;
            padding: 0.5rem 1rem;
            border-radius: 0.75rem;
            font-weight: 600;
            border: 2px solid #d1d5db;
        }
        .timer-buttons button:hover {
            background-color: #e5e7eb;
        }
        .timer-buttons button.active {
            background-color: #10b981;
            color: white;
            border-color: #10b981;
        }
    </style>
</head>
<body class="bg-gray-100 flex items-center justify-center min-h-screen">

<div class="container space-y-8">
    <h1 class="text-4xl font-bold text-center text-gray-800">Quiz Generator</h1>

    <!-- Quiz Creation Section -->
    <div id="quiz-editor" class="bg-gray-50 p-6 rounded-xl space-y-6">
        <h2 class="text-2xl font-bold text-gray-700">Create Your Quiz</h2>
        <div class="space-y-4">
            <div>
                <label for="quiz-title" class="block text-sm font-medium text-gray-700">Quiz Title</label>
                <input type="text" id="quiz-title" placeholder="e.g., Entrepreneurship Quiz" class="input-field mt-1">
            </div>
            <div>
                <label for="quiz-content" class="block text-sm font-medium text-gray-700">Quiz Questions & Answers</label>
                <textarea id="quiz-content" rows="10" placeholder="Paste your quiz questions here. Use a clear format like:
Q1. Question text?
A. Correct Answer
B. Incorrect Answer
C. Incorrect Answer

Q2. Second question?
A. Correct Answer" class="input-field mt-1 resize-y"></textarea>
            </div>
            <div>
                <label for="answer-key" class="block text-sm font-medium text-gray-700">Answer Key</label>
                <textarea id="answer-key" rows="3" placeholder="Paste your answer key here. Separate with commas or new lines.
e.g., 1. A, 2. B, 3. C" class="input-field mt-1 resize-y"></textarea>
            </div>
            <div class="flex justify-end">
                <button id="create-quiz-btn" class="btn-primary">Generate Quiz</button>
            </div>
        </div>
    </div>

    <!-- Timer Section -->
    <div id="timer-section" class="bg-gray-50 p-6 rounded-xl space-y-6 hidden">
        <h2 class="text-2xl font-bold text-gray-700">Quiz Timer</h2>
        <div class="flex flex-col items-center space-y-4">
            <div class="timer-buttons flex flex-wrap justify-center gap-2">
                <button data-time="300" class="rounded-lg p-2 transition-all duration-300 transform hover:scale-105">5 min</button>
                <button data-time="600" class="rounded-lg p-2 transition-all duration-300 transform hover:scale-105">10 min</button>
                <button data-time="900" class="rounded-lg p-2 transition-all duration-300 transform hover:scale-105">15 min</button>
                <button data-time="1800" class="rounded-lg p-2 transition-all duration-300 transform hover:scale-105">30 min</button>
                <button data-time="3600" class="rounded-lg p-2 transition-all duration-300 transform hover:scale-105">1 hr</button>
            </div>
            <div id="custom-timer-group" class="flex items-center space-x-2">
                <input type="number" id="custom-minutes" placeholder="Custom mins" class="input-field w-32 text-center">
                <button id="start-custom-timer-btn" class="btn-primary">Set Timer</button>
            </div>
            <div id="timer-display" class="timer-display">00:00</div>
        </div>
    </div>
    
    <!-- Quiz Display Section -->
    <div id="quiz-display" class="bg-gray-50 p-6 rounded-xl space-y-6 hidden">
        <h2 id="quiz-display-title" class="text-2xl font-bold text-gray-700"></h2>
        <div id="quiz-questions" class="space-y-4"></div>
        <div class="flex justify-end">
            <button id="submit-quiz-btn" class="btn-primary">Submit Quiz</button>
        </div>
        <div id="results" class="mt-6 p-4 rounded-xl bg-gray-100 hidden">
            <h3 class="text-xl font-bold text-gray-700">Quiz Results</h3>
            <p id="score-text" class="mt-2 text-lg text-gray-800"></p>
            <div id="result-details" class="mt-4 space-y-2"></div>
        </div>
    </div>
</div>

<script>
    const quizTitleInput = document.getElementById('quiz-title');
    const quizContentInput = document.getElementById('quiz-content');
    const answerKeyInput = document.getElementById('answer-key');
    const createQuizBtn = document.getElementById('create-quiz-btn');
    const quizEditor = document.getElementById('quiz-editor');
    const timerSection = document.getElementById('timer-section');
    const quizDisplay = document.getElementById('quiz-display');
    const quizDisplayTitle = document.getElementById('quiz-display-title');
    const quizQuestionsContainer = document.getElementById('quiz-questions');
    const submitQuizBtn = document.getElementById('submit-quiz-btn');
    const resultsContainer = document.getElementById('results');
    const scoreText = document.getElementById('score-text');
    const resultDetails = document.getElementById('result-details');

    const timerButtons = document.querySelectorAll('.timer-buttons button');
    const customMinutesInput = document.getElementById('custom-minutes');
    const startCustomTimerBtn = document.getElementById('start-custom-timer-btn');
    const timerDisplay = document.getElementById('timer-display');
    
    let quizData = {};
    let timerInterval;
    let timeRemaining = 0;

    // --- Quiz Generation Logic ---
    createQuizBtn.addEventListener('click', () => {
        const title = quizTitleInput.value;
        const content = quizContentInput.value;
        const answerKey = answerKeyInput.value;

        if (!title || !content || !answerKey) {
            alert('Please fill in all fields to generate the quiz.');
            return;
        }

        quizData = parseQuizContent(content, answerKey);
        
        quizEditor.classList.add('hidden');
        timerSection.classList.remove('hidden');
        quizDisplay.classList.remove('hidden');
        quizDisplayTitle.textContent = title;
        renderQuiz();
    });

    function parseQuizContent(content, key) {
        const lines = content.split('\n').filter(line => line.trim() !== '');
        const questions = [];
        let currentQuestion = null;

        lines.forEach(line => {
            if (line.match(/^Q\d+\./)) {
                if (currentQuestion) {
                    questions.push(currentQuestion);
                }
                currentQuestion = {
                    text: line.substring(line.indexOf('.') + 1).trim(),
                    options: []
                };
            } else if (line.match(/^[A-Z]\./) && currentQuestion) {
                currentQuestion.options.push({
                    text: line.substring(line.indexOf('.') + 1).trim(),
                    label: line.charAt(0)
                });
            }
        });
        if (currentQuestion) {
            questions.push(currentQuestion);
        }

        const answerKeyMap = {};
        key.split(/[,.\n]/).forEach(item => {
            const parts = item.trim().split('.');
            if (parts.length === 2) {
                const qNum = parseInt(parts[0].trim());
                const answer = parts[1].trim().toUpperCase();
                if (!isNaN(qNum) && answer.length === 1) {
                    answerKeyMap[qNum] = answer;
                }
            }
        });

        return { questions, answerKey: answerKeyMap };
    }

    function renderQuiz() {
        quizQuestionsContainer.innerHTML = '';
        quizData.questions.forEach((q, index) => {
            const questionElement = document.createElement('div');
            questionElement.classList.add('question', 'p-4', 'bg-white', 'rounded-xl', 'border-2', 'border-gray-200');
            questionElement.innerHTML = `<p class="font-semibold text-lg text-gray-800">Q${index + 1}. ${q.text}</p>`;

            const optionsList = document.createElement('div');
            optionsList.classList.add('mt-2', 'space-y-2');

            q.options.forEach(option => {
                const optionElement = document.createElement('label');
                optionElement.classList.add('block', 'p-3', 'rounded-xl', 'bg-gray-100', 'cursor-pointer', 'hover:bg-gray-200', 'transition-colors', 'duration-200', 'flex', 'items-center', 'space-x-2');
                optionElement.innerHTML = `
                    <input type="radio" name="q${index}" value="${option.label}" class="form-radio text-blue-600 h-4 w-4">
                    <span class="text-gray-700">${option.text}</span>
                `;
                optionsList.appendChild(optionElement);
            });

            questionElement.appendChild(optionsList);
            quizQuestionsContainer.appendChild(questionElement);
        });
    }

    // --- Timer Logic ---
    function startTimer() {
        if (timerInterval) clearInterval(timerInterval);
        
        updateTimerDisplay();
        
        timerInterval = setInterval(() => {
            timeRemaining--;
            updateTimerDisplay();
            if (timeRemaining <= 0) {
                clearInterval(timerInterval);
                alert('Time is up! The quiz will be submitted automatically.');
                submitQuiz();
            }
        }, 1000);
    }
    
    function updateTimerDisplay() {
        const minutes = Math.floor(timeRemaining / 60);
        const seconds = timeRemaining % 60;
        timerDisplay.textContent = `${String(minutes).padStart(2, '0')}:${String(seconds).padStart(2, '0')}`;
    }

    timerButtons.forEach(button => {
        button.addEventListener('click', () => {
            timerButtons.forEach(btn => btn.classList.remove('active'));
            button.classList.add('active');
            timeRemaining = parseInt(button.dataset.time, 10);
            startTimer();
        });
    });

    startCustomTimerBtn.addEventListener('click', () => {
        const minutes = parseInt(customMinutesInput.value, 10);
        if (!isNaN(minutes) && minutes > 0) {
            timerButtons.forEach(btn => btn.classList.remove('active'));
            timeRemaining = minutes * 60;
            startTimer();
        } else {
            alert('Please enter a valid number of minutes.');
        }
    });

    // --- Quiz Submission Logic ---
    submitQuizBtn.addEventListener('click', () => {
        clearInterval(timerInterval);
        submitQuiz();
    });

    function submitQuiz() {
        let score = 0;
        let correctCount = 0;
        resultDetails.innerHTML = '';

        quizData.questions.forEach((q, index) => {
            const questionNumber = index + 1;
            const selectedAnswer = document.querySelector(`input[name="q${index}"]:checked`);
            const correctAnswer = quizData.answerKey[questionNumber];
            
            const resultElement = document.createElement('div');
            resultElement.classList.add('p-3', 'rounded-lg');
            
            const isCorrect = selectedAnswer && selectedAnswer.value === correctAnswer;
            if (isCorrect) {
                score++;
                correctCount++;
                resultElement.classList.add('bg-green-100');
                resultElement.innerHTML = `<span class="font-semibold text-green-700">Q${questionNumber}. Correct!</span> Your answer was ${selectedAnswer.value}.`;
            } else {
                resultElement.classList.add('bg-red-100');
                const userAnswer = selectedAnswer ? selectedAnswer.value : 'No answer';
                resultElement.innerHTML = `<span class="font-semibold text-red-700">Q${questionNumber}. Incorrect.</span> Your answer was ${userAnswer}. The correct answer was ${correctAnswer}.`;
            }

            resultDetails.appendChild(resultElement);
        });

        scoreText.textContent = `You scored ${correctCount} out of ${quizData.questions.length}!`;
        resultsContainer.classList.remove('hidden');
    }

</script>

</body>
</html>
