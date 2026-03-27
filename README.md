<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Wizarding World Personality Quiz</title>

<style>
@import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@500&family=EB+Garamond&display=swap');

body {
    margin: 0;
    font-family: 'EB Garamond', serif;
    background: radial-gradient(circle at top, #1a1a2e, #0f0f1a);
    color: #f5e6c8;
}

/* magical stars */
body::before {
    content:"";
    position:fixed;
    width:200%;
    height:200%;
    background-image: radial-gradient(#ffffff22 1px, transparent 1px);
    background-size:40px 40px;
    animation:stars 60s linear infinite;
}
@keyframes stars {
    from {transform:translateY(0);}
    to {transform:translateY(-100px);}
}

.container {
    max-width:900px;
    margin:40px auto;
    background:rgba(20,20,30,0.92);
    border:1px solid gold;
    border-radius:12px;
    padding:25px;
    box-shadow:0 0 25px black;
    position:relative;
    z-index:1;
}

h1 {
    font-family:'Cinzel', serif;
    text-align:center;
    color:gold;
}

.question {
    margin-bottom:18px;
    padding:12px;
    border-left:3px solid gold;
    transition:0.2s;
}
.question:hover { transform:scale(1.02); }

label { display:block; margin:6px 0; cursor:pointer; }
label:hover { color:gold; }

button {
    display:block;
    margin:25px auto;
    padding:12px 20px;
    background:linear-gradient(45deg,gold,#8b6f1d);
    border:none;
    color:black;
    font-weight:bold;
    border-radius:6px;
    cursor:pointer;
}
button:hover { box-shadow:0 0 12px gold; }

.result {
    margin-top:25px;
    padding:20px;
    background:rgba(0,0,0,0.6);
    border-left:5px solid gold;
    border-radius:8px;
    animation:fadeIn 1s ease;
}
@keyframes fadeIn {
    from{opacity:0; transform:translateY(10px);}
    to{opacity:1; transform:translateY(0);}
}

/* bars */
.bar {
    height:8px;
    background:#333;
    margin:6px 0;
    border-radius:4px;
}
.fill {
    height:100%;
    background:gold;
    width:0%;
    transition:width 1s ease;
}

/* character card */
.card {
    margin-top:20px;
    padding:15px;
    background:#111;
    border:1px solid gold;
    border-radius:10px;
    display:flex;
    gap:15px;
    align-items:center;
}
.card img {
    width:120px;
    height:120px;
    object-fit:cover;
    border-radius:8px;
    border:1px solid gold;
}
</style>
</head>

<body>

<div class="container">
<h1>🧙 Wizarding Personality Assessment</h1>
<p>Choose one statement from each group:</p>

<form id="quizForm"></form>

<button onclick="calculateResult()">Reveal My Magical Identity</button>

<div id="result" class="result" style="display:none;"></div>
</div>

<script>
// QUESTIONS (UNCHANGED)
const questions = [
["Tackles demanding goals with urgency and sustained energy","Seeks feedback proactively and applies it quickly","Inspires and mobilizes others around a compelling objective"],
["Maintains poise and clarity when stakes are high","Shares credit freely and remains modest about achievements","Upholds principles and chooses the right path over the easy one"],
["Pushes through obstacles to deliver results ahead of schedule","Commands attention through confident communication","Welcomes critique and turns it into improvements"],
["Aligns teams behind a common vision","Demonstrates modesty during success","Acts with transparency and honesty"],
["Drives momentum and meets timelines","Acknowledges others’ contributions","Projects composure in formal settings"],
["Adapts rapidly after guidance","Guides peers and builds commitment","Chooses ethical action under pressure"],
["Works relentlessly to surpass goals","Incorporates feedback to improve","Exudes confidence socially"],
["Rallies others to achieve goals","Credits teammates first","Holds high moral standards"]
];

// TRAITS
const traitsMap = [
["drive","growth","leader"],
["leader","humble","moral"],
["drive","leader","growth"],
["leader","humble","moral"],
["drive","humble","leader"],
["growth","leader","moral"],
["drive","growth","leader"],
["drive","humble","moral"]
];

// CHARACTER DATABASE (deep cuts + lore + image)
const characters = {
"kingsley":{
name:"Kingsley Shacklebolt",
house:"Gryffindor",
patronus:"Lynx",
img:"https://upload.wikimedia.org/wikipedia/en/6/65/Kingsley_Shacklebolt.jpg",
desc:"A calm, authoritative leader who commands respect without needing to demand it."
},
"regulus":{
name:"Regulus Black",
house:"Slytherin",
patronus:"Unknown (likely serpent-like)",
img:"https://upload.wikimedia.org/wikipedia/en/0/0c/RegulusBlack.jpg",
desc:"A quiet moral turner—someone who changes course when it truly matters."
},
"sprout":{
name:"Pomona Sprout",
house:"Hufflepuff",
patronus:"Badger",
img:"https://upload.wikimedia.org/wikipedia/en/4/4f/Pomona_Sprout.jpg",
desc:"A nurturing leader who builds strength in others without seeking recognition."
},
"bill":{
name:"Bill Weasley",
house:"Gryffindor",
patronus:"Wolf",
img:"https://upload.wikimedia.org/wikipedia/en/8/8e/Bill_Weasley.jpg",
desc:"Adventurous, capable, and quietly excellent in everything he does."
},
"doge":{
name:"Elphias Doge",
house:"Ravenclaw",
patronus:"Unknown",
img:"https://upload.wikimedia.org/wikipedia/en/6/6e/Elphias_Doge.jpg",
desc:"Loyal and principled, the kind of person who stands by others no matter what."
},
"ted":{
name:"Ted Tonks",
house:"Hufflepuff",
patronus:"Weasel",
img:"https://upload.wikimedia.org/wikipedia/en/5/5b/TedTonks.jpg",
desc:"Kind, grounded, and quietly brave in the face of injustice."
},
"neville":{
name:"Neville Longbottom",
house:"Gryffindor",
patronus:"Non-corporeal (later lion-like courage)",
img:"https://upload.wikimedia.org/wikipedia/en/2/2f/Neville_Longbottom.jpg",
desc:"Growth through persistence—becoming formidable without losing humility."
},
"charlie":{
name:"Charlie Weasley",
house:"Gryffindor",
patronus:"Dragon",
img:"https://upload.wikimedia.org/wikipedia/en/3/36/Charlie_Weasley.jpg",
desc:"A natural leader who thrives in challenging, unpredictable environments."
}
};

// BUILD FORM
const form = document.getElementById("quizForm");
questions.forEach((q,i)=>{
    let div=document.createElement("div");
    div.className="question";
    div.innerHTML=`<h3>Question ${i+1}</h3>`;
    q.forEach((text,j)=>{
        div.innerHTML+=`
        <label>
        <input type="radio" name="q${i}" value="${traitsMap[i][j]}" required>
        ${text}
        </label>`;
    });
    form.appendChild(div);
});

// RESULT LOGIC
function calculateResult(){
let scores={drive:0,growth:0,leader:0,humble:0,moral:0};
let data=new FormData(form);
for(let v of data.values()){ scores[v]++; }

// determine archetype
let sorted=Object.entries(scores).sort((a,b)=>b[1]-a[1]);
let combo=sorted[0][0]+"-"+sorted[1][0];

let mapping={
"drive-leader":"kingsley",
"growth-moral":"regulus",
"leader-humble":"sprout",
"drive-growth":"bill",
"leader-moral":"doge",
"humble-moral":"ted",
"growth-humble":"neville",
"growth-leader":"charlie"
};

let charKey=mapping[combo]||mapping[sorted[1][0]+"-"+sorted[0][0]]||"kingsley";
let c=characters[charKey];

// display
let resultDiv=document.getElementById("result");
resultDiv.style.display="block";

resultDiv.innerHTML=`
<h2>Your Magical Profile</h2>

<p><b>House:</b> ${c.house}</p>
<p><b>Patronus:</b> ${c.patronus}</p>

<h3>Trait Balance</h3>
${Object.keys(scores).map(t=>`
<div>${t}</div>
<div class="bar"><div class="fill" style="width:${scores[t]*20}%"></div></div>
`).join("")}

<div class="card">
<img src="${c.img}">
<div>
<h3>${c.name}</h3>
<p>${c.desc}</p>
</div>
</div>
`;
}
</script>

</body>
</html>
