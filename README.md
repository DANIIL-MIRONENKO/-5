// ================= Утиліти =================
const $ = sel => document.querySelector(sel);

function shuffle(arr){
  for(let i = arr.length - 1; i > 0; i--){
    const j = Math.floor(Math.random() * (i+1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }
  return arr;
}

function formatTime(sec){
  const m = Math.floor(sec/60);
  const s = sec % 60;
  return `${m}:${s < 10 ? "0"+s : s}`;
}

// ================= Клас питання =================
class Question {
  constructor(text, answers, correct){
    this.text = text;
    this.answers = shuffle([...answers]);
    this.correct = correct;
  }

  display(){
    questionEl.textContent = this.text;
    answersEl.innerHTML = "";
    this.answers.forEach(ans=>{
      const btn = document.createElement("button");
      btn.textContent = ans;
      answersEl.appendChild(btn);
    });
  }
}

// ================= DOM =================
const questionEl = $("#question");
const answersEl  = $("#answers");
const quizEl     = $("#quiz");
const startBtn   = $("#startBtn");
const statsEl    = $("#stats");
const startWrap  = $(".start-container");
const timerEl    = $("#timer");
const progressEl = $("#progress");

// ================= Дані =================
let currentIndex = 0;
let correctCount = 0;
let totalCount   = 0;
let currentQuestion = null;
let timeLeft = 180;
let timerId = null;

// ================= 40 питань про Іспанію =================
const questions = [
  new Question("Столиця Іспанії?", ["Мадрид","Барселона","Севілья","Валенсія"], "Мадрид"),
  new Question("Яка національна страва Іспанії?", ["Паелья","Тако","Піцца","Кускус"], "Паелья"),
  new Question("Офіційна мова Іспанії?", ["Іспанська","Каталонська","Баскська","Галісійська"], "Іспанська"),
  new Question("Яке місто відоме собором Саграда Фамілія?", ["Барселона","Мадрид","Гранада","Кордова"], "Барселона"),
  new Question("Найвища гора Іспанії?", ["Тейде","Анаето","Мулясен","Монблан"], "Тейде"),
  new Question("Який іспанський художник написав 'Герніку'?", ["Пабло Пікассо","Сальвадор Далі","Франсиско Гойя","Дієго Веласкес"], "Пабло Пікассо"),
  new Question("Яке місто відоме своїми коридами?", ["Памплона","Севілья","Мадрид","Толедо"], "Памплона"),
  new Question("Який танець вважається традиційним в Іспанії?", ["Фламенко","Танго","Самба","Сальса"], "Фламенко"),
  new Question("Яке море омиває східне узбережжя Іспанії?", ["Середземне","Атлантичний океан","Чорне","Балтійське"], "Середземне"),
  new Question("Який відомий футбольний клуб базується у Барселоні?", ["Барселона","Реал Мадрид","Атлетіко","Валенсія"], "Барселона"),
  new Question("Яка валюта використовується в Іспанії?", ["Євро","Песо","Песета","Фунт"], "Євро"),
  new Question("Як називається відомий іспанський холодний суп?", ["Гаспачо","Борщ","Мінестроне","Суп місо"], "Гаспачо"),
  new Question("Яке місто відоме Альгамброю?", ["Гранада","Севілья","Мадрид","Барселона"], "Гранада"),
  new Question("Який іспанський художник був представником сюрреалізму?", ["Сальвадор Далі","Пікассо","Міро","Гойя"], "Сальвадор Далі"),
  new Question("Скільки автономних спільнот в Іспанії?", ["17","15","20","12"], "17"),
  new Question("Який архіпелаг належить Іспанії і знаходиться в Атлантичному океані?", ["Канарські острови","Балеарські острови","Мадейра","Азори"], "Канарські острови"),
  new Question("Яка річка найдовша в Іспанії?", ["Ебро","Тахо","Гвадалквівір","Дуеро"], "Тахо"),
  new Question("Яке місто відоме кафедральним собором і 'Білим місяцем'?", ["Толедо","Севілья","Кордова","Бургос"], "Толедо"),
  new Question("Який відомий фестиваль томатів проводять в Іспанії?", ["Ла Томатіна","Сан-Фермін","Фальяс","Семана Санта"], "Ла Томатіна"),
  new Question("Який футбольний клуб називають 'королівським'?", ["Реал Мадрид","Барселона","Атлетіко","Севілья"], "Реал Мадрид"),
  new Question("Який океан омиває західне узбережжя Іспанії?", ["Атлантичний","Індійський","Середземне море","Червоне"], "Атлантичний"),
  new Question("Яке місто відоме собором Хіронімо?", ["Сантьяго-де-Компостела","Севілья","Толедо","Бургос"], "Сантьяго-де-Компостела"),
  new Question("Яка іспанська провінція відома вином Ріоха?", ["Ла-Ріоха","Андалусія","Каталонія","Кастилія"], "Ла-Ріоха"),
  new Question("Яке місто відоме як центр каталонської культури?", ["Барселона","Мадрид","Валенсія","Сарагоса"], "Барселона"),
  new Question("Яка пустеля найбільша в Європі і знаходиться в Іспанії?", ["Табернас","Сахара","Каракуми","Гобі"], "Табернас"),
  new Question("Яка автономна область відома баскською мовою?", ["Країна Басків","Каталонія","Андалусія","Галісія"], "Країна Басків"),
  new Question("Яке іспанське місто називають 'Містом мистецтв і наук'?", ["Валенсія","Мадрид","Севілья","Барселона"], "Валенсія"),
  new Question("Який іспанський святий є покровителем країни?", ["Святий Яків","Святий Георгій","Святий Петро","Святий Павло"], "Святий Яків"),
  new Question("Яка традиційна страва з картоплі та яєць популярна в Іспанії?", ["Тортилья","Омлет","Фрітата","Шакшука"], "Тортилья"),
  new Question("Яке місто відоме як центр фламенко?", ["Севілья","Мадрид","Барселона","Валенсія"], "Севілья"),
  new Question("Який іспанський художник був відомий чорними картинами?", ["Франсиско Гойя","Далі","Веласкес","Міро"], "Франсиско Гойя"),
  new Question("Яке місто є столицею Андалусії?", ["Севілья","Гранада","Малага","Кордова"], "Севілья"),
  new Question("Який архіпелаг належить Іспанії у Середземному морі?", ["Балеарські острови","Канарські","Азори","Кіпр"], "Балеарські острови"),
  new Question("Яке місто відоме Рамблою?", ["Барселона","Мадрид","Валенсія","Сарагоса"], "Барселона"),
  new Question("Яка іспанська команда виграла найбільше Ліги чемпіонів?", ["Реал Мадрид","Барселона","Атлетіко","Севілья"], "Реал Мадрид"),
  new Question("Яка гора розділяє Іспанію та Францію?", ["Піренеї","Альпи","Апенніни","Карпати"], "Піренеї"),
  new Question("Який відомий іспанський напій із вина та фруктів?", ["Сангрія","Шампанське","Пунш","Калімочо"], "Сангрія"),
  new Question("Яке місто відоме Кордовською мечеттю?", ["Кордова","Севілья","Толедо","Гранада"], "Кордова"),
  new Question("Який іспанський письменник написав 'Дон Кіхота'?", ["Мігель де Сервантес","Гарсія Лорка","Кальдерон","Кеведо"], "Мігель де Сервантес"),
  new Question("Яке іспанське місто називають 'Воротами Африки'?", ["Альхесірас","Малага","Севілья","Барселона"], "Альхесірас")
];

// ================= Логіка з таймером =================
function updateProgress(){
  progressEl.textContent = `Питання ${Math.min(currentIndex+1, questions.length)} з ${questions.length}`;
}

function startTimer(){
  clearInterval(timerId);
  timeLeft = 180;
  timerEl.textContent = `Час: ${formatTime(timeLeft)}`;

  timerId = setInterval(()=>{
    timeLeft--;
    timerEl.textContent = `Час: ${formatTime(timeLeft)}`;
    if(timeLeft <= 0){
      clearInterval(timerId);
      totalCount++;
      showCorrectAnswer();
      setTimeout(()=>{
        currentIndex++;
        newQuestion();
      }, 1000);
    }
  },1000);
}

function stopTimer(){
  clearInterval(timerId);
  timerId = null;
}

function showCorrectAnswer(){
  [...answersEl.querySelectorAll("button")].forEach(btn=>{
    if(btn.textContent === currentQuestion.correct){
      anime({ targets: btn, backgroundColor: '#8BC34A', duration: 500 });
    }
    btn.disabled = true;
  });
}

function newQuestion(){
  if(currentIndex >= questions.length){
    endQuiz();
    return;
  }
  currentQuestion = questions[currentIndex];
  currentQuestion.display();
  updateProgress();
  startTimer();

  [...answersEl.querySelectorAll("button")].forEach(btn=>{
    btn.addEventListener("click", ()=>{
      totalCount++;
      stopTimer();

      const isCorrect = btn.textContent === currentQuestion.correct;
      if(isCorrect){
        correctCount++;
        anime({ targets: btn, backgroundColor: '#8BC34A', duration: 350 });
      } else {
        anime({ targets: btn, backgroundColor: '#F44336', duration: 350 });
        showCorrectAnswer();
      }

      [...answersEl.querySelectorAll("button")].forEach(b=>b.disabled = true);

      setTimeout(()=>{
        currentIndex++;
        newQuestion();
      }, 1000);
    });
  });
}

function startQuiz(){
  currentIndex = 0;
  correctCount = 0;
  totalCount   = 0;
  statsEl.textContent = "";
  startWrap.classList.add("hidden");
  quizEl.classList.remove("hidden");
  newQuestion();
}

function endQuiz(){
  stopTimer();
  quizEl.classList.add("hidden");
  startWrap.classList.remove("hidden");

  const accuracy = totalCount ? Math.round(correctCount*100/totalCount) : 0;
  statsEl.textContent = `Ви дали ${correctCount} правильних відповідей із ${totalCount}. Точність — ${accuracy}%`;
  alert(`Правильно: ${correctCount}\nУсього: ${totalCount}\nТочність: ${accuracy}%`);
}

// ================= Події =================
startBtn.addEventListener("click", startQuiz);
