# Gestion-caisse
import React, { useState, useEffect } from ‘react’;
import { Camera, Image, TrendingUp, DollarSign, Calendar, Plus, Trash2, CreditCard, Smartphone, Banknote, Lock, LogOut } from ‘lucide-react’;
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from ‘recharts’;

export default function CashRegisterApp() {
const [isAuthenticated, setIsAuthenticated] = useState(false);
const [loginForm, setLoginForm] = useState({ identifiant: ‘’, code: ‘’ });
const [activeTab, setActiveTab] = useState(‘recettes’);
const [recettes, setRecettes] = useState([]);
const [fondsCaisse, setFondsCaisse] = useState([]);
const [showAddForm, setShowAddForm] = useState(false);
const [formData, setFormData] = useState({
montant: ‘’,
date: new Date().toISOString().split(‘T’)[0],
note: ‘’,
categorie: ‘especes’
});

// Identifiants de connexion - À MODIFIER PAR VOS PROPRES IDENTIFIANTS
const CREDENTIALS = {
identifiant: ‘admin’, // Changez ceci
code: ‘1234’ // Changez ceci
};

useEffect(() => {
// Vérifier si l’utilisateur est déjà connecté
const auth = localStorage.getItem(‘isAuthenticated’);
if (auth === ‘true’) {
setIsAuthenticated(true);
loadData();
}
}, []);

const handleLogin = (e) => {
e.preventDefault();
if (loginForm.identifiant === CREDENTIALS.identifiant && loginForm.code === CREDENTIALS.code) {
setIsAuthenticated(true);
localStorage.setItem(‘isAuthenticated’, ‘true’);
loadData();
} else {
alert(‘Identifiant ou code incorrect’);
setLoginForm({ identifiant: ‘’, code: ‘’ });
}
};

const handleLogout = () => {
setIsAuthenticated(false);
localStorage.removeItem(‘isAuthenticated’);
setLoginForm({ identifiant: ‘’, code: ‘’ });
};

const loadData = () => {
const savedRecettes = localStorage.getItem(‘recettes’);
const savedFonds = localStorage.getItem(‘fondsCaisse’);
if (savedRecettes) setRecettes(JSON.parse(savedRecettes));
if (savedFonds) setFondsCaisse(JSON.parse(savedFonds));
};

const saveData = (newRecettes, newFonds) => {
localStorage.setItem(‘recettes’, JSON.stringify(newRecettes));
localStorage.setItem(‘fondsCaisse’, JSON.stringify(newFonds));
};

const handlePhotoCapture = async (e) => {
const file = e.target.files[0];
if (!file) return;

```
try {
  const Tesseract = window.Tesseract;
  if (!Tesseract) {
    alert("Le module de reconnaissance de texte n'est pas chargé. Veuillez réessayer.");
    return;
  }

  const { data: { text } } = await Tesseract.recognize(file, 'fra');
  
  const numbers = text.match(/\d+[.,]?\d*/g);
  if (numbers && numbers.length > 0) {
    const amount = numbers[0].replace(',', '.');
    setFormData({ ...formData, montant: amount });
  } else {
    alert("Aucun nombre détecté dans l'image");
  }
} catch (error) {
  console.error("Erreur de reconnaissance:", error);
  alert("Erreur lors de la lecture de l'image. Veuillez saisir manuellement.");
}
```

};

const addEntry = () => {
if (!formData.montant || isNaN(parseFloat(formData.montant))) {
alert(“Veuillez entrer un montant valide”);
return;
}

```
const newEntry = {
  id: Date.now(),
  montant: parseFloat(formData.montant),
  date: formData.date,
  note: formData.note,
  categorie: formData.categorie
};

if (activeTab === 'recettes') {
  const newRecettes = [...recettes, newEntry];
  setRecettes(newRecettes);
  saveData(newRecettes, fondsCaisse);
} else {
  const newFonds = [...fondsCaisse, newEntry];
  setFondsCaisse(newFonds);
  saveData(recettes, newFonds);
}

setFormData({ 
  montant: '', 
  date: new Date().toISOString().split('T')[0], 
  note: '',
  categorie: 'especes'
});
setShowAddForm(false);
```

};

const deleteEntry = (id) => {
if (!confirm(‘Êtes-vous sûr de vouloir supprimer cette entrée ?’)) return;

```
if (activeTab === 'recettes') {
  const newRecettes = recettes.filter(r => r.id !== id);
  setRecettes(newRecettes);
  saveData(newRecettes, fondsCaisse);
} else {
  const newFonds = fondsCaisse.filter(f => f.id !== id);
  setFondsCaisse(newFonds);
  saveData(recettes, newFonds);
}
```

};

const getCurrentDayTotal = (data) => {
const today = new Date().toISOString().split(‘T’)[0];
return data
.filter(item => item.date === today)
.reduce((sum, item) => sum + item.montant, 0);
};

const getCurrentMonthTotal = (data) => {
const currentMonth = new Date().getMonth();
const currentYear = new Date().getFullYear();
return data
.filter(item => {
const itemDate = new Date(item.date);
return itemDate.getMonth() === currentMonth && itemDate.getFullYear() === currentYear;
})
.reduce((sum, item) => sum + item.montant, 0);
};

const getCurrentMonthTotalByCategory = (data, categorie) => {
const currentMonth = new Date().getMonth();
const currentYear = new Date().getFullYear();
return data
.filter(item => {
const itemDate = new Date(item.date);
return itemDate.getMonth() === currentMonth &&
itemDate.getFullYear() === currentYear &&
item.categorie === categorie;
})
.reduce((sum, item) => sum + item.montant, 0);
};

const getCurrentDayTotalByCategory = (data, categorie) => {
const today = new Date().toISOString().split(‘T’)[0];
return data
.filter(item => item.date === today && item.categorie === categorie)
.reduce((sum, item) => sum + item.montant, 0);
};

const getChartData = (data) => {
const grouped = {};
data.forEach(item => {
const date = item.date;
if (!grouped[date]) {
grouped[date] = { especes: 0, cb: 0, borne: 0 };
}
grouped[date][item.categorie] = (grouped[date][item.categorie] || 0) + item.montant;
});

```
return Object.keys(grouped)
  .sort()
  .slice(-30)
  .map(date => ({
    date: new Date(date).toLocaleDateString('fr-FR', { day: '2-digit', month: '2-digit' }),
    especes: grouped[date].especes,
    cb: grouped[date].cb,
    borne: grouped[date].borne
  }));
```

};

const getCategoryIcon = (categorie) => {
switch(categorie) {
case ‘especes’: return <Banknote size={16} />;
case ‘cb’: return <CreditCard size={16} />;
case ‘borne’: return <Smartphone size={16} />;
default: return null;
}
};

const getCategoryLabel = (categorie) => {
switch(categorie) {
case ‘especes’: return ‘Espèces’;
case ‘cb’: return ‘CB’;
case ‘borne’: return ‘Borne’;
default: return categorie;
}
};

// Page de connexion
if (!isAuthenticated) {
return (
<div className="min-h-screen bg-gradient-to-br from-blue-500 to-indigo-600 flex items-center justify-center p-4">
<div className="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-md">
<div className="text-center mb-8">
<div className="bg-blue-100 w-20 h-20 rounded-full flex items-center justify-center mx-auto mb-4">
<Lock size={40} className="text-blue-600" />
</div>
<h1 className="text-3xl font-bold text-gray-800 mb-2">Gestion de Caisse</h1>
<p className="text-gray-600">Connectez-vous pour continuer</p>
</div>

```
      <form onSubmit={handleLogin} className="space-y-4">
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">
            Identifiant
          </label>
          <input
            type="text"
            value={loginForm.identifiant}
            onChange={(e) => setLoginForm({ ...loginForm, identifiant: e.target.value })}
            className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            placeholder="Entrez votre identifiant"
            required
          />
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">
            Code
          </label>
          <input
            type="password"
            value={loginForm.code}
            onChange={(e) => setLoginForm({ ...loginForm, code: e.target.value })}
            className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            placeholder="Entrez votre code"
            required
          />
        </div>

        <button
          type="submit"
          className="w-full bg-blue-600 hover:bg-blue-700 text-white py-3 rounded-lg font-semibold transition shadow-lg"
        >
          Se connecter
        </button>
      </form>
```

None
</div>
</div>
);
}

// Application principale
const currentData = activeTab === ‘recettes’ ? recettes : fondsCaisse;
const dayTotal = getCurrentDayTotal(currentData);
const monthTotal = getCurrentMonthTotal(currentData);
const monthEspeces = getCurrentMonthTotalByCategory(currentData, ‘especes’);
const monthCB = getCurrentMonthTotalByCategory(currentData, ‘cb’);
const monthBorne = getCurrentMonthTotalByCategory(currentData, ‘borne’);
const dayEspeces = getCurrentDayTotalByCategory(currentData, ‘especes’);
const dayCB = getCurrentDayTotalByCategory(currentData, ‘cb’);
const dayBorne = getCurrentDayTotalByCategory(currentData, ‘borne’);
const chartData = getChartData(currentData);

return (
<div className="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-4">
<div className="max-w-4xl mx-auto">
<div className="flex items-center justify-between mb-6">
<h1 className="text-3xl font-bold text-gray-800">
💰 Gestion de Caisse
</h1>
<button
onClick={handleLogout}
className="flex items-center gap-2 px-4 py-2 bg-red-500 hover:bg-red-600 text-white rounded-lg font-medium transition shadow"
>
<LogOut size={18} />
Déconnexion
</button>
</div>

```
    {/* Onglets */}
    <div className="flex gap-2 mb-6">
      <button
        onClick={() => setActiveTab('recettes')}
        className={`flex-1 py-3 px-4 rounded-lg font-semibold flex items-center justify-center gap-2 transition ${
          activeTab === 'recettes'
            ? 'bg-green-500 text-white shadow-lg'
            : 'bg-white text-gray-600 hover:bg-gray-50'
        }`}
      >
        <TrendingUp size={20} />
        Recettes
      </button>
      <button
        onClick={() => setActiveTab('fondsCaisse')}
        className={`flex-1 py-3 px-4 rounded-lg font-semibold flex items-center justify-center gap-2 transition ${
          activeTab === 'fondsCaisse'
            ? 'bg-blue-500 text-white shadow-lg'
            : 'bg-white text-gray-600 hover:bg-gray-50'
        }`}
      >
        <DollarSign size={20} />
        Fond de Caisse
      </button>
    </div>

    {/* Carte Total du Jour et du Mois */}
    <div className={`${activeTab === 'recettes' ? 'bg-green-500' : 'bg-blue-500'} rounded-xl p-6 text-white mb-4 shadow-lg`}>
      <div className="flex items-center justify-between mb-4">
        <div>
          <p className="text-sm opacity-90 mb-1">Total du jour</p>
          <p className="text-3xl font-bold">{dayTotal.toFixed(2)} €</p>
        </div>
      </div>
      <div className="flex items-center justify-between border-t border-white/20 pt-4">
        <div>
          <p className="text-sm opacity-90 mb-1">Total du mois</p>
          <p className="text-2xl font-bold">{monthTotal.toFixed(2)} €</p>
        </div>
        <Calendar size={40} className="opacity-50" />
      </div>
    </div>

    {/* Cartes par catégorie */}
    <div className="grid grid-cols-3 gap-3 mb-6">
      <div className="bg-white rounded-lg p-4 shadow">
        <div className="flex items-center gap-2 text-gray-600 mb-2">
          <Banknote size={18} />
          <span className="text-sm font-medium">Espèces</span>
        </div>
        <p className="text-xl font-bold text-gray-800">{dayEspeces.toFixed(2)} €</p>
        <p className="text-xs text-gray-500 mt-1">Mois: {monthEspeces.toFixed(2)} €</p>
      </div>
      <div className="bg-white rounded-lg p-4 shadow">
        <div className="flex items-center gap-2 text-gray-600 mb-2">
          <CreditCard size={18} />
          <span className="text-sm font-medium">CB</span>
        </div>
        <p className="text-xl font-bold text-gray-800">{dayCB.toFixed(2)} €</p>
        <p className="text-xs text-gray-500 mt-1">Mois: {monthCB.toFixed(2)} €</p>
      </div>
      <div className="bg-white rounded-lg p-4 shadow">
        <div className="flex items-center gap-2 text-gray-600 mb-2">
          <Smartphone size={18} />
          <span className="text-sm font-medium">Borne</span>
        </div>
        <p className="text-xl font-bold text-gray-800">{dayBorne.toFixed(2)} €</p>
        <p className="text-xs text-gray-500 mt-1">Mois: {monthBorne.toFixed(2)} €</p>
      </div>
    </div>

    {/* Graphique */}
    {chartData.length > 0 && (
      <div className="bg-white rounded-xl p-6 mb-6 shadow-lg">
        <h3 className="text-lg font-semibold text-gray-700 mb-4">Évolution (30 derniers jours)</h3>
        <ResponsiveContainer width="100%" height={250}>
          <LineChart data={chartData}>
            <CartesianGrid strokeDasharray="3 3" />
            <XAxis dataKey="date" />
            <YAxis />
            <Tooltip formatter={(value) => `${value.toFixed(2)} €`} />
            <Legend />
            <Line type="monotone" dataKey="especes" stroke="#10b981" strokeWidth={2} name="Espèces" />
            <Line type="monotone" dataKey="cb" stroke="#3b82f6" strokeWidth={2} name="CB" />
            <Line type="monotone" dataKey="borne" stroke="#8b5cf6" strokeWidth={2} name="Borne" />
          </LineChart>
        </ResponsiveContainer>
      </div>
    )}

    {/* Bouton Ajouter */}
    <button
      onClick={() => setShowAddForm(!showAddForm)}
      className={`w-full ${activeTab === 'recettes' ? 'bg-green-500 hover:bg-green-600' : 'bg-blue-500 hover:bg-blue-600'} text-white py-4 rounded-xl font-semibold flex items-center justify-center gap-2 mb-6 shadow-lg transition`}
    >
      <Plus size={24} />
      Ajouter une entrée
    </button>

    {/* Formulaire d'ajout */}
    {showAddForm && (
      <div className="bg-white rounded-xl p-6 mb-6 shadow-lg">
        <h3 className="text-lg font-semibold text-gray-700 mb-4">Nouvelle entrée</h3>
        
        <label className="block mb-4">
          <span className="text-sm font-medium text-gray-700 mb-2 block">Ajouter une photo</span>
          <div className="grid grid-cols-2 gap-2">
            <input
              type="file"
              accept="image/*"
              capture="environment"
              onChange={handlePhotoCapture}
              className="hidden"
              id="photo-camera"
            />
            <label
              htmlFor="photo-camera"
              className="flex items-center justify-center gap-2 bg-gray-100 border-2 border-dashed border-gray-300 rounded-lg p-4 cursor-pointer hover:bg-gray-200 transition"
            >
              <Camera size={20} className="text-gray-600" />
              <span className="text-sm text-gray-600">Appareil photo</span>
            </label>

            <input
              type="file"
              accept="image/*"
              onChange={handlePhotoCapture}
              className="hidden"
              id="photo-gallery"
            />
            <label
              htmlFor="photo-gallery"
              className="flex items-center justify-center gap-2 bg-gray-100 border-2 border-dashed border-gray-300 rounded-lg p-4 cursor-pointer hover:bg-gray-200 transition"
            >
              <Image size={20} className="text-gray-600" />
              <span className="text-sm text-gray-600">Galerie</span>
            </label>
          </div>
        </label>

        <label className="block mb-4">
          <span className="text-sm font-medium text-gray-700 mb-2 block">Catégorie</span>
          <div className="grid grid-cols-3 gap-2">
            <button
              type="button"
              onClick={() => setFormData({ ...formData, categorie: 'especes' })}
              className={`p-3 rounded-lg border-2 flex flex-col items-center gap-1 transition ${
                formData.categorie === 'especes'
                  ? 'border-green-500 bg-green-50 text-green-700'
                  : 'border-gray-300 bg-white text-gray-600 hover:bg-gray-50'
              }`}
            >
              <Banknote size={24} />
              <span className="text-xs font-medium">Espèces</span>
            </button>
            <button
              type="button"
              onClick={() => setFormData({ ...formData, categorie: 'cb' })}
              className={`p-3 rounded-lg border-2 flex flex-col items-center gap-1 transition ${
                formData.categorie === 'cb'
                  ? 'border-blue-500 bg-blue-50 text-blue-700'
                  : 'border-gray-300 bg-white text-gray-600 hover:bg-gray-50'
              }`}
            >
              <CreditCard size={24} />
              <span className="text-xs font-medium">CB</span>
            </button>
            <button
              type="button"
              onClick={() => setFormData({ ...formData, categorie: 'borne' })}
              className={`p-3 rounded-lg border-2 flex flex-col items-center gap-1 transition ${
                formData.categorie === 'borne'
                  ? 'border-purple-500 bg-purple-50 text-purple-700'
                  : 'border-gray-300 bg-white text-gray-600 hover:bg-gray-50'
              }`}
            >
              <Smartphone size={24} />
              <span className="text-xs font-medium">Borne</span>
            </button>
          </div>
        </label>

        <label className="block mb-4">
          <span className="text-sm font-medium text-gray-700 mb-2 block">Montant (€)</span>
          <input
            type="number"
            step="0.01"
            value={formData.montant}
            onChange={(e) => setFormData({ ...formData, montant: e.target.value })}
            className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            placeholder="0.00"
          />
        </label>

        <label className="block mb-4">
          <span className="text-sm font-medium text-gray-700 mb-2 block">Date</span>
          <input
            type="date"
            value={formData.date}
            onChange={(e) => setFormData({ ...formData, date: e.target.value })}
            className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
          />
        </label>

        <label className="block mb-4">
          <span className="text-sm font-medium text-gray-700 mb-2 block">Note (optionnel)</span>
          <input
            type="text"
            value={formData.note}
            onChange={(e) => setFormData({ ...formData, note: e.target.value })}
            className="w-full px-4 py-3 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-500 focus:border-transparent"
            placeholder="Description..."
          />
        </label>

        <div className="flex gap-2">
          <button
            onClick={addEntry}
            className={`flex-1 ${activeTab === 'recettes' ? 'bg-green-500 hover:bg-green-600' : 'bg-blue-500 hover:bg-blue-600'} text-white py-3 rounded-lg font-semibold transition`}
          >
            Ajouter
          </button>
          <button
            onClick={() => setShowAddForm(false)}
            className="flex-1 bg-gray-200 hover:bg-gray-300 text-gray-700 py-3 rounded-lg font-semibold transition"
          >
            Annuler
          </button>
        </div>
      </div>
    )}

    {/* Liste des entrées */}
    <div className="bg-white rounded-xl p-6 shadow-lg">
      <h3 className="text-lg font-semibold text-gray-700 mb-4">Historique</h3>
      {currentData.length === 0 ? (
        <p className="text-gray-500 text-center py-8">Aucune entrée pour le moment</p>
      ) : (
        <div className="space-y-2">
          {currentData.sort((a, b) => new Date(b.date) - new Date(a.date)).map(item => (
            <div key={item.id} className="flex items-center justify-between p-4 bg-gray-50 rounded-lg hover:bg-gray-100 transition">
              <div className="flex-1">
                <div className="flex items-center gap-2 mb-1">
                  {getCategoryIcon(item.categorie)}
                  <span className="text-xs font-medium text-gray-500">{getCategoryLabel(item.categorie)}</span>
                </div>
                <p className="font-semibold text-gray-800">{item.montant.toFixed(2)} €</p>
                <p className="text-sm text-gray-600">
                  {new Date(item.date).toLocaleDateString('fr-FR')}
                  {item.note && ` • ${item.note}`}
                </p>
              </div>
              <button
                onClick={() => deleteEntry(item.id)}
                className="text-red-500 hover:text-red-700 p-2 hover:bg-red-50 rounded-lg transition"
              >
                <Trash2 size={20} />
              </button>
            </div>
          ))}
        </div>
      )}
    </div>
  </div>

  <script src="https://cdnjs.cloudflare.com/ajax/libs/tesseract.js/4.1.1/tesseract.min.js"></script>
</div>
```

);
}