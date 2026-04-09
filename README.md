[Index.html.txt](https://github.com/user-attachments/files/26615106/Index.html.txt)
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>NPM Field App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <style>
        :root { --forest: #1a2e05; --grass: #3e5622; --dirt: #c5a059; }
        body { background-color: #f8f8f8; font-family: -apple-system, system-ui, sans-serif; -webkit-tap-highlight-color: transparent; }
        .sun-card { @apply bg-white rounded-3xl shadow-sm border border-stone-200 p-5 mb-4 active:scale-[0.98] transition-all; }
        .input-field { @apply w-full p-4 rounded-xl border-2 border-stone-200 bg-white text-lg mb-3 focus:border-[#3e5622] outline-none font-medium; }
        .btn-primary { @apply bg-[#1a2e05] text-white font-black py-4 px-6 rounded-2xl shadow-lg active:bg-black w-full text-center; }
        .nav-btn { @apply flex flex-col items-center justify-center w-full py-3; }
        .modal-overlay { @apply fixed inset-0 bg-black/70 backdrop-blur-sm z-[100] flex items-end justify-center; }
        .modal-content { @apply bg-white w-full max-w-md rounded-t-[3rem] p-8 animate-in slide-in-from-bottom duration-300 pb-12; }
    </style>
</head>
<body>
    <div id="root"></div>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
        import { getAuth, signInWithEmailAndPassword, signOut, onAuthStateChanged, setPersistence, browserLocalPersistence } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, doc, updateDoc, deleteDoc, orderBy } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

        const firebaseConfig = {
            apiKey: "AIzaSyAbPZO0PBMojki9axxrB5b30WqiVh4jgsE",
            authDomain: "npm-app-3dbc7.firebaseapp.com",
            projectId: "npm-app-3dbc7",
            storageBucket: "npm-app-3dbc7.firebasestorage.app",
            messagingSenderId: "714816852874",
            appId: "1:714816852874:web:e569408de4b487fd66ee73"
        };
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        setPersistence(auth, browserLocalPersistence);
        window.NPM_FB = { auth, db, addDoc, collection, query, onSnapshot, doc, updateDoc, deleteDoc, signInWithEmailAndPassword, signOut, orderBy };
    </script>

    <script type="text/babel">
        const { useState, useEffect } = React;
        const fb = window.NPM_FB;
        const SERVICES = ["Stump Grinding", "Land Clearing", "Auger Services", "Storm Cleanup", "Debris Hauling", "Free Estimate", "Other"];

        const App = () => {
            const [user, setUser] = useState(null);
            const [view, setView] = useState('dashboard');
            const [customers, setCustomers] = useState([]);
            const [appointments, setAppointments] = useState([]);
            const [isAddingJob, setIsAddingJob] = useState(null);

            useEffect(() => {
                const unsubAuth = fb.onAuthStateChanged(fb.auth, u => setUser(u));
                if (user) {
                    const unsubCust = fb.onSnapshot(fb.query(fb.collection(fb.db, "customers"), fb.orderBy("name")), (s) => 
                        setCustomers(s.docs.map(d => ({id: d.id, ...d.data()})))
                    );
                    const unsubAppt = fb.onSnapshot(fb.query(fb.collection(fb.db, "appointments"), fb.orderBy("date")), (s) => 
                        setAppointments(s.docs.map(d => ({id: d.id, ...d.data()})))
                    );
                    return () => { unsubCust(); unsubAppt(); };
                }
            }, [user]);

            if (!user) return <LoginScreen />;

            return (
                <div className="max-w-md mx-auto min-h-screen pb-24">
                    <div className="p-6 bg-[#1a2e05] text-white flex justify-between items-center rounded-b-[2.5rem] shadow-xl">
                        <div><h1 className="text-2xl font-black italic">NPM</h1><p className="text-[10px] font-bold opacity-60 uppercase">Nance Property Maintenance</p></div>
                        <button onClick={() => fb.signOut(fb.auth)} className="bg-white/10 px-3 py-1 rounded-full text-[10px] font-black">EXIT</button>
                    </div>

                    <main className="p-4">
                        {view === 'dashboard' && <Dashboard appointments={appointments} />}
                        {view === 'customers' && <CustomersList customers={customers} onAddJob={setIsAddingJob} />}
                        {view === 'appointments' && <AppointmentsList appointments={appointments} />}
                        {view === 'reminders' && <RemindersView appointments={appointments} customers={customers} />}
                    </main>

                    <nav className="fixed bottom-0 left-0 right-0 bg-white border-t border-stone-200 flex justify-around pb-10 pt-3 z-50 shadow-2xl">
                        <NavIcon active={view==='dashboard'} label="Home" icon="🏠" onClick={()=>setView('dashboard')} />
                        <NavIcon active={view==='appointments'} label="Jobs" icon="📅" onClick={()=>setView('appointments')} />
                        <NavIcon active={view==='customers'} label="Clients" icon="👥" onClick={()=>setView('customers')} />
                        <NavIcon active={view==='reminders'} label="Remind" icon="🔔" onClick={()=>setView('reminders')} />
                    </nav>

                    {isAddingJob && <JobModal customer={isAddingJob} onClose={()=>setIsAddingJob(null)} />}
                </div>
            );
        };

        const NavIcon = ({ active, label, icon, onClick }) => (
            <button onClick={onClick} className={`flex flex-col items-center w-full ${active ? 'text-[#3e5622]' : 'text-stone-300'}`}>
                <span className="text-xl mb-1">{icon}</span>
                <span className="text-[9px] font-black uppercase">{label}</span>
            </button>
        );

        const Dashboard = ({ appointments }) => {
            const today = new Date().toISOString().split('T')[0];
            const jobsToday = appointments.filter(a => a.date.includes(today));
            return (
                <div className="animate-in fade-in">
                    <div className="sun-card bg-[#3e5622] text-white border-none shadow-lg">
                        <p className="font-bold opacity-70 uppercase text-[10px] tracking-widest">Active Schedule</p>
                        <h2 className="text-4xl font-black">{jobsToday.length} Jobs Today</h2>
                    </div>
                    <h3 className="font-black text-[#1a2e05] mt-6 mb-4 flex items-center gap-2"><span>📍</span> ON THE DOCKET</h3>
                    {jobsToday.length === 0 ? <p className="text-stone-400 italic text-center py-10">Clear for today.</p> :
                        jobsToday.map(j => (
                            <div key={j.id} className="sun-card flex justify-between items-center border-l-8 border-[#c5a059]">
                                <div><div className="font-black text-lg leading-tight">{j.service}</div><div className="text-stone-500 font-bold text-sm">{j.customerName}</div></div>
                                <div className="font-black text-[#3e5622] text-lg">${j.price}</div>
                            </div>
                        ))
                    }
                </div>
            );
        };

        const CustomersList = ({ customers, onAddJob }) => {
            const [q, setQ] = useState('');
            const addCustomer = async () => {
                const name = prompt("Name:"); const phone = prompt("Phone:");
                if(name && phone) await fb.addDoc(fb.collection(fb.db, "customers"), { name, phone });
            };
            return (
                <div className="animate-in fade-in">
                    <div className="flex gap-2 mb-4">
                        <input className="input-field mb-0" placeholder="Search..." onChange={e=>setQ(e.target.value.toLowerCase())} />
                        <button onClick={addCustomer} className="bg-[#c5a059] text-white px-6 rounded-xl font-black">+</button>
                    </div>
                    {customers.filter(c => c.name.toLowerCase().includes(q)).map(c => (
                        <div key={c.id} className="sun-card">
                            <div className="flex justify-between items-center mb-4">
                                <div><h4 className="font-black text-xl leading-tight">{c.name}</h4><p className="text-stone-400 font-bold">{c.phone}</p></div>
                                <button onClick={()=>onAddJob(c)} className="bg-[#1a2e05] text-white text-[10px] font-black px-4 py-3 rounded-xl shadow-md">NEW JOB</button>
                            </div>
                            <div className="flex gap-2">
                                <a href={`tel:${c.phone}`} className="flex-1 bg-stone-100 py-3 rounded-xl text-center text-lg">📞</a>
                                <a href={`sms:${c.phone}`} className="flex-1 bg-stone-100 py-3 rounded-xl text-center text-lg">💬</a>
                            </div>
                        </div>
                    ))}
                </div>
            );
        };

        const AppointmentsList = ({ appointments }) => (
            <div className="animate-in fade-in">
                {appointments.map(a => (
                    <div key={a.id} className={`sun-card ${a.status==='Completed' ? 'opacity-40 grayscale' : ''}`}>
                        <div className="flex justify-between text-[10px] font-black uppercase text-[#c5a059] mb-1">
                            <span>{new Date(a.date).toLocaleString([], {month:'short', day:'numeric', hour:'2-digit', minute:'2-digit'})}</span>
                            <span className="bg-stone-100 px-2 py-0.5 rounded text-stone-600">{a.status}</span>
                        </div>
                        <h4 className="text-xl font-black text-stone-800 leading-tight">{a.service}</h4>
                        <p className="font-bold text-stone-500 mb-4">{a.customerName}</p>
                        <div className="flex gap-2">
                            <button onClick={() => fb.updateDoc(fb.doc(fb.db, "appointments", a.id), { status: 'Completed' })} className="flex-1 py-3 bg-stone-800 text-white rounded-xl font-black text-xs">COMPLETE</button>
                            <button onClick={() => {if(confirm("Delete?")) fb.deleteDoc(fb.doc(fb.db, "appointments", a.id))}} className="px-4 py-3 bg-red-50 text-red-600 rounded-xl">🗑️</button>
                        </div>
                    </div>
                ))}
            </div>
        );

        const RemindersView = ({ appointments, customers }) => {
            const send = (a) => {
                const c = customers.find(x => x.id === a.customerId);
                const msg = `Hi ${a.customerName}, this is Nance Property Maintenance. Friendly reminder of your ${a.service} on ${new Date(a.date).toLocaleString([], {month:'short', day:'numeric', hour:'2-digit', minute:'2-digit'})}. See you soon!`;
                window.open(`sms:${c.phone}?&body=${encodeURIComponent(msg)}`);
                fb.updateDoc(fb.doc(fb.db, "appointments", a.id), { reminderSent: true });
            };
            return (
                <div className="animate-in fade-in">
                    {appointments.filter(a => !a.reminderSent && a.status !== 'Completed').map(a => (
                        <div key={a.id} className="sun-card border-l-8 border-[#c5a059]">
                            <p className="font-black text-stone-800 text-lg">{a.customerName}</p>
                            <p className="text-xs font-bold text-stone-400 mb-4 uppercase tracking-tighter">{a.service}</p>
                            <button onClick={()=>send(a)} className="btn-primary">SEND TEXT REMINDER</button>
                        </div>
                    ))}
                </div>
            );
        };

        const JobModal = ({ customer, onClose }) => {
            const [service, setService] = useState(SERVICES[0]);
            const [date, setDate] = useState('');
            const [price, setPrice] = useState('');
            const save = async () => {
                if(!date || !price) return alert("Missing Info");
                await fb.addDoc(fb.collection(fb.db, "appointments"), {
                    customerId: customer.id, customerName: customer.name,
                    service, date, price, status: 'Scheduled', reminderSent: false
                });
                onClose();
            };
            return (
                <div className="modal-overlay" onClick={onClose}>
                    <div className="modal-content" onClick={e=>e.stopPropagation()}>
                        <div className="w-12 h-1.5 bg-stone-200 rounded-full mx-auto mb-6"></div>
                        <h3 className="text-2xl font-black mb-6 text-[#1a2e05]">Job for {customer.name}</h3>
                        <label className="text-[10px] font-black uppercase text-stone-400 ml-1">Work Type</label>
                        <select className="input-field" onChange={e=>setService(e.target.value)}>
                            {SERVICES.map(s => <option key={s} value={s}>{s}</option>)}
                        </select>
                        <label className="text-[10px] font-black uppercase text-stone-400 ml-1">Schedule</label>
                        <input type="datetime-local" className="input-field" onChange={e=>setDate(e.target.value)} />
                        <label className="text-[10px] font-black uppercase text-stone-400 ml-1">Price</label>
                        <input type="number" className="input-field" placeholder="0.00" onChange={e=>setPrice(e.target.value)} />
                        <button onClick={save} className="btn-primary mt-4 py-5 text-lg">CONFIRM & SAVE</button>
                    </div>
                </div>
            );
        };

        const LoginScreen = () => {
            const [e, setE] = useState(''); const [p, setP] = useState('');
            return (
                <div className="p-10 flex flex-col justify-center min-h-screen">
                    <div className="mb-10 text-center"><h1 className="text-6xl font-black text-[#1a2e05] italic">NPM</h1><p className="font-bold text-stone-400 uppercase tracking-widest text-xs">Property Maintenance</p></div>
                    <input className="input-field" placeholder="Email" onChange={x=>setE(x.target.value)} />
                    <input className="input-field" type="password" placeholder="Password" onChange={x=>setP(x.target.value)} />
                    <button onClick={()=>fb.signInWithEmailAndPassword(fb.auth, e, p)} className="btn-primary mt-4">LOG IN</button>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
