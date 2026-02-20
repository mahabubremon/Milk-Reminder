<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Milk Business Pro - Permanent Dark Mode</title>

<script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

<style>
/* 🌑 Full Dark Theme CSS */
*{box-sizing:border-box; font-family: 'Segoe UI', Arial, sans-serif;}

body { 
    margin:0; 
    background: #0f172a; /* Deep Dark Background */
    color: #e2e8f0; 
    min-height: 100vh; 
    padding-bottom: 50px; 
}

.container { max-width:600px; margin:auto; padding:15px; }

.card { 
    background: rgba(30, 41, 59, 0.7); 
    padding: 20px; 
    margin: 15px 0; 
    border-radius: 16px; 
    box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1);
    border: 1px solid rgba(255, 255, 255, 0.1);
}

h2, h3 { text-align: center; color: #38bdf8; margin-top: 0; }

input, select { 
    width: 100%; 
    padding: 12px; 
    margin: 8px 0; 
    border: 1px solid #334155; 
    border-radius: 8px; 
    font-size: 16px; 
    background: #1e293b; 
    color: white; 
}

input:focus { outline: 2px solid #38bdf8; border-color: transparent; }

button { 
    width: 100%; 
    padding: 12px; 
    margin-top: 10px; 
    border: none; 
    border-radius: 8px; 
    background: #38bdf8; 
    color: #0f172a;
    font-weight: bold; 
    cursor: pointer; 
    font-size: 16px;
    transition: 0.3s;
}

button:hover { background: #7dd3fc; }

.small-red { background: #ef4444; color: white; margin-top: 15px; }

.history-table { 
    width: 100%; 
    border-collapse: collapse; 
    margin-top: 10px; 
    font-size: 13px; 
}

.history-table th, .history-table td { 
    padding: 12px; 
    text-align: left; 
    border-bottom: 1px solid #334155; 
}

.history-table th { color: #94a3b8; font-weight: 500; }

.del-btn { 
    background: #7f1d1d; 
    color: #fecaca; 
    border: none; 
    padding: 5px 10px; 
    border-radius: 5px; 
    cursor: pointer; 
}

.due-item {
    display: flex; 
    justify-content: space-between; 
    background: #1e293b; 
    padding: 12px; 
    border-radius: 10px; 
    margin: 8px 0;
    border-left: 4px solid #38bdf8;
}

.total-due-text {
    font-size: 20px;
    text-align: center;
    color: #fbbf24;
    font-weight: bold;
}
</style>
</head>
<body>

<div class="container">

    <div id="loginPage" class="card">
        <h2>🥛 Milk Pro Dark</h2>
        <input type="text" id="email" placeholder="Username">
        <input type="password" id="password" placeholder="Password">
        <button onclick="login()">প্রবেশ করুন (Login)</button>
    </div>

    <div id="appPage" style="display:none">

        <div class="card" style="text-align: right;">
            <span style="float: left; color: #94a3b8;">User: Remon1200</span>
            <button class="small-red" style="width: auto; padding: 5px 15px;" onclick="logout()">Logout</button>
        </div>

        <div class="card">
            <h3>👤 কাস্টমার যোগ করুন</h3>
            <input type="text" id="custName" placeholder="কাস্টমারের নাম">
            <button onclick="addCustomer()">কাস্টমার সেভ করুন</button>
        </div>

        <div class="card">
            <h3>📝 এন্ট্রি / টাকা জমা</h3>
            <label style="font-size: 12px; color: #94a3b8;">তারিখ:</label>
            <input type="date" id="entryDate">
            <select id="customerList"></select>
            <input type="number" id="milk" placeholder="দুধ (লিটার) - না নিলে ০">
            <input type="number" id="price" placeholder="লিটার প্রতি দাম (৳)" value="70">
            <input type="number" id="paid" placeholder="জমা দেওয়া টাকা (৳)">
            <button onclick="addEntry()">হিসাব সেভ করুন</button>
        </div>

        <div class="card">
            <h3>💰 বর্তমান বকেয়া (Due List)</h3>
            <div id="dueList"></div>
            <div class="total-due-text">মোট বকেয়া: ৳<span id="totalDue">0</span></div>
        </div>

        <div class="card">
            <h3>📜 হিস্ট্রি ও ডিলিট</h3>
            <div style="overflow-x:auto;">
                <table class="history-table">
                    <thead>
                        <tr>
                            <th>তারিখ</th>
                            <th>নাম</th>
                            <th>লিটার</th>
                            <th>জমা</th>
                            <th>অ্যাকশন</th>
                        </tr>
                    </thead>
                    <tbody id="historyBody"></tbody>
                </table>
            </div>
        </div>

    </div>
</div>

<script>
// লগইন ক্রেডেনশিয়াল
const fixedUser = "Remon1200";
const fixedPass = "Remon@1200";

// আজকের তারিখ অটো সেট
document.getElementById('entryDate').valueAsDate = new Date();

let customers = JSON.parse(localStorage.getItem('milk_customers')) || {};
let entries = JSON.parse(localStorage.getItem('milk_entries')) || [];

function login(){
    if(email.value === fixedUser && password.value === fixedPass){
        loginPage.style.display="none";
        appPage.style.display="block";
        localStorage.setItem("loggedIn","yes");
        loadAll();
    } else { alert("ভুল পাসওয়ার্ড!"); }
}

function logout(){ localStorage.removeItem("loggedIn"); location.reload(); }

window.onload = function(){
    if(localStorage.getItem("loggedIn")==="yes"){
        loginPage.style.display="none";
        appPage.style.display="block";
        loadAll();
    }
}

function addCustomer(){
    let name = custName.value.trim();
    if(!name) return alert("নাম লিখুন");
    let id = "cust_" + Date.now();
    customers[id] = {name: name, due: 0};
    saveData();
    custName.value="";
    loadAll();
}

function loadAll(){
    loadCustomers();
    loadDueList();
    loadHistory();
}

function loadCustomers(){
    customerList.innerHTML="";
    for(let id in customers){
        let opt = document.createElement("option");
        opt.value=id; opt.text=customers[id].name;
        customerList.appendChild(opt);
    }
}

function addEntry(){
    let id = customerList.value;
    let date = document.getElementById('entryDate').value;
    let milkVal = parseFloat(milk.value||0);
    let priceVal = parseFloat(price.value||0);
    let paidVal = parseFloat(paid.value||0);
    
    if(!id || !date) return alert("কাস্টমার ও তারিখ সিলেক্ট করুন");

    let totalCost = milkVal * priceVal;
    let balanceChange = totalCost - paidVal;

    entries.push({
        entryId: Date.now(),
        id: id,
        name: customers[id].name,
        date: date,
        milk: milkVal,
        paid: paidVal,
        diff: balanceChange
    });

    customers[id].due += balanceChange;
    saveData();
    
    milk.value=""; paid.value="";
    loadAll();
}

function loadHistory(){
    let body = document.getElementById('historyBody');
    body.innerHTML = "";
    [...entries].reverse().slice(0, 20).forEach((item) => {
        body.innerHTML += `<tr>
            <td>${item.date}</td>
            <td>${item.name}</td>
            <td>${item.milk}L</td>
            <td>৳${item.paid}</td>
            <td><button class="del-btn" onclick="removeEntry(${item.entryId})">মুছুন</button></td>
        </tr>`;
    });
}

function removeEntry(entryId){
    if(confirm("এই এন্ট্রিটি মুছে ফেললে বকেয়া হিসাব আগের মতো হয়ে যাবে। নিশ্চিত?")){
        let index = entries.findIndex(e => e.entryId === entryId);
        if(index > -1){
            let item = entries[index];
            if(customers[item.id]){ customers[item.id].due -= item.diff; }
            entries.splice(index, 1);
            saveData();
            loadAll();
        }
    }
}

function loadDueList(){
    let total=0;
    dueList.innerHTML="";
    for(let id in customers){
        let data=customers[id];
        dueList.innerHTML+= `<div class="due-item">
            <span>${data.name}</span> <b>৳${data.due.toFixed(2)}</b>
        </div>`;
        total+=data.due;
    }
    totalDue.innerText = total.toFixed(2);
}

function saveData(){
    localStorage.setItem('milk_customers', JSON.stringify(customers));
    localStorage.setItem('milk_entries', JSON.stringify(entries));
}
</script>

</body>
</html>
