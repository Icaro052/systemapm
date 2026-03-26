import React, { useState, useEffect, useMemo } from 'react';
import { 
  Calendar, 
  ChevronLeft, 
  ChevronRight, 
  CheckCircle2,
  Clock,
  Settings,
  Download,
  Search,
  X,
  Activity,
  BarChart3,
  TrendingUp,
  Cloud,
  CloudOff,
  Loader2,
  Timer,
  FileDown,
  ShieldAlert,
  ShieldCheck
} from 'lucide-react';

// --- CONFIGURAÇÃO FIREBASE ---
import { initializeApp } from 'firebase/app';
import { getAuth, signInWithCustomToken, signInAnonymously, onAuthStateChanged } from 'firebase/auth';
import { getDatabase, ref, set, remove, onValue } from 'firebase/database';

// Inicialização segura do Firebase compatível com o ambiente
let app, auth, db, appId, safeAppId;
const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : null;

if (firebaseConfig) {
  try {
    app = initializeApp(firebaseConfig);
    auth = getAuth(app);
    db = getDatabase(app);
    appId = typeof __app_id !== 'undefined' ? __app_id : 'heineken-pmday-alagoinhas';
    // Remove caracteres inválidos (., #, $, [, ]) que o Realtime Database não aceita nas chaves principais
    safeAppId = appId.replace(/[.#$\[\]]/g, '_');
  } catch (e) {
    console.error("Erro ao inicializar Firebase", e);
  }
}

const DB_PATH = 'pmdays_producao';
const ADMINS_PATH = 'admins';
const USERS_PATH = 'usuarios';

// Função auxiliar para calcular o número da semana (W)
const getWeekNumber = (date) => {
  const d = new Date(Date.UTC(date.getFullYear(), date.getMonth(), date.getDate()));
  const dayNum = d.getUTCDay() || 7;
  d.setUTCDate(d.getUTCDate() + 4 - dayNum);
  const yearStart = new Date(Date.UTC(d.getUTCFullYear(),0,1));
  return Math.ceil((((d - yearStart) / 86400000) + 1)/7);
};

const App = () => {
  // Estados da Aplicação
  const [isLoggedIn, setIsLoggedIn] = useState(false);
  const [isRegistering, setIsRegistering] = useState(false);
  const [loginEmail, setLoginEmail] = useState(''); 
  const [registerName, setRegisterName] = useState(''); // Captura o nome
  const [loggedEmail, setLoggedEmail] = useState(''); 
  
  const [currentDate, setCurrentDate] = useState(new Date(2026, 2, 1)); // Março 2026
  const [selectedCell, setSelectedCell] = useState(null);
  const [isDrawerOpen, setIsDrawerOpen] = useState(false);
  const [searchTerm, setSearchTerm] = useState('');
  const [currentTime, setCurrentTime] = useState(new Date());
  
  // Estados de Dados e Firebase
  const [user, setUser] = useState(null);
  const [pmData, setPmData] = useState([]);
  const [adminsList, setAdminsList] = useState([]); 
  const [isSyncing, setIsSyncing] = useState(true);
  const [syncError, setSyncError] = useState(false);

  const linhas = [
    "Utilidades",
    "Processo",
    "Linha 01",
    "Linha 02",
    "Linha 03",
    "Linha 04 PET",
    "Linha 05 PET",
    "Linha 08",
    "Linha 09",
    "Linha 10",
    "Linha 11",
    "Linha 12"
  ];

  // Verifica se o utilizador atual tem privilégios de Administrador
  const isUserAdmin = adminsList.includes(loggedEmail.toLowerCase()) || adminsList.length === 0;

  // Relógio em tempo real
  useEffect(() => {
    const timer = setInterval(() => setCurrentTime(new Date()), 1000);
    return () => clearInterval(timer);
  }, []);

  // --- EFEITOS DE BASE DE DADOS (FIREBASE AUTH & SYNC) ---
  useEffect(() => {
    const initAuth = async () => {
      if (!auth) {
        setSyncError(true);
        setIsSyncing(false);
        return;
      }
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Erro na autenticação:", error);
        setSyncError(true);
        setIsSyncing(false);
      }
    };
    
    initAuth();

    if (auth) {
      const unsubscribe = onAuthStateChanged(auth, (currentUser) => {
        setUser(currentUser);
        if (!currentUser) setIsSyncing(false);
      });
      return () => unsubscribe();
    }
  }, []);

  useEffect(() => {
    if (!user || syncError || !db || !safeAppId) return;

    setIsSyncing(true);
    
    // Referência aos dados de PMDay
    const pmdaysRef = ref(db, `artifacts/${safeAppId}/public/data/${DB_PATH}`);
    const unsubPmdays = onValue(pmdaysRef, (snapshot) => {
      const data = [];
      snapshot.forEach((childSnapshot) => {
        data.push({ id: childSnapshot.key, ...childSnapshot.val() });
      });
      setPmData(data);
      setIsSyncing(false);
      setSyncError(false);
    }, (error) => {
      console.error("Erro ao sincronizar dados:", error);
      setSyncError(true);
      setIsSyncing(false);
    });

    // Referência à lista de Administradores
    const adminsRef = ref(db, `artifacts/${safeAppId}/public/data/${ADMINS_PATH}`);
    const unsubAdmins = onValue(adminsRef, (snapshot) => {
      if (snapshot.exists()) {
        const adminsData = snapshot.val();
        setAdminsList(Object.values(adminsData).map(e => e.toLowerCase()));
      } else {
        setAdminsList([]);
      }
    });

    return () => {
      unsubPmdays();
      unsubAdmins();
    };
  }, [user, syncError]);

  // --- LOGIN SUBMIT E REGISTO DE UTILIZADOR NO FIREBASE ---
  const handleLoginSubmit = async (e) => {
    e.preventDefault();
    const emailToLogin = loginEmail.toLowerCase();
    setLoggedEmail(emailToLogin);
    setIsLoggedIn(true);

    if (!db || !safeAppId) return;

    // Chave segura para o Firebase (substitui pontos por underscore)
    const safeEmailKey = emailToLogin.replace(/[.#$\[\]]/g, '_');

    try {
      // 1. Grava o perfil do utilizador na base de dados (Aparecerá no seu Firebase!)
      const userRef = ref(db, `artifacts/${safeAppId}/public/data/${USERS_PATH}/${safeEmailKey}`);
      await set(userRef, {
        email: emailToLogin,
        nome: isRegistering ? registerName : emailToLogin.split('@')[0],
        ultimoAcesso: new Date().toISOString(),
        tipoConta: (adminsList.length === 0 || adminsList.includes(emailToLogin)) ? 'Administrador' : 'Visualizador'
      });

      // 2. Se a base de dados estiver vazia de admins, o primeiro a entrar torna-se Admin
      if (adminsList.length === 0) {
        const newAdminRef = ref(db, `artifacts/${safeAppId}/public/data/${ADMINS_PATH}/${Date.now()}`);
        await set(newAdminRef, emailToLogin);
      }
    } catch (err) {
      console.error("Erro ao gravar utilizador no Firebase:", err);
    }
  };

  // --- CÁLCULOS E LÓGICA DE INTERFACE ---
  const filteredLinhas = useMemo(() => {
    if (!searchTerm) return linhas;
    return linhas.filter(l => l.toLowerCase().includes(searchTerm.toLowerCase()));
  }, [searchTerm, linhas]);

  const metrics = useMemo(() => {
    const currentMonthStr = `${currentDate.getFullYear()}-${String(currentDate.getMonth() + 1).padStart(2, '0')}`;
    const pmsThisMonth = pmData.filter(pm => pm.dataStr && pm.dataStr.startsWith(currentMonthStr));
    
    const total = pmsThisMonth.length;
    const realizados = pmsThisMonth.filter(pm => pm.status === 'realizado').length;
    const programados = pmsThisMonth.filter(pm => pm.status === 'programado').length;
    const taxa = total === 0 ? 0 : Math.round((realizados / total) * 100);
    const horasTotais = pmsThisMonth.reduce((acc, pm) => acc + (Number(pm.duracao) || 0), 0);

    return { total, realizados, programados, taxa, horasTotais };
  }, [pmData, currentDate]);

  const monthData = useMemo(() => {
    const year = currentDate.getFullYear();
    const month = currentDate.getMonth();
    const date = new Date(year, month, 1);
    const weeksMap = new Map();

    while (date.getMonth() === month) {
      const dayOfWeek = date.getDay();
      if (dayOfWeek !== 0 && dayOfWeek !== 6) { 
        const weekNum = getWeekNumber(date);
        const dayInfo = {
          dayNumber: date.getDate(),
          dateStr: `${year}-${String(month + 1).padStart(2, '0')}-${String(date.getDate()).padStart(2, '0')}`,
          weekDayName: date.toLocaleDateString('pt-PT', { weekday: 'short' }).replace('.', '').substring(0, 3)
        };

        if (!weeksMap.has(weekNum)) {
          weeksMap.set(weekNum, { weekNumber: weekNum, days: [] });
        }
        weeksMap.get(weekNum).days.push(dayInfo);
      }
      date.setDate(date.getDate() + 1);
    }
    return Array.from(weeksMap.values());
  }, [currentDate]);

  const getCellData = (linha, dateStr) => {
    return pmData.find(p => p.linha === linha && p.dataStr === dateStr);
  };

  const handleCellClick = (linha, dayInfo) => {
    if (!isUserAdmin) {
      alert("Acesso Negado: Apenas administradores do sistema podem modificar o planeamento do PMDay.");
      return;
    }
    const existing = getCellData(linha, dayInfo.dateStr);
    setSelectedCell({ linha, dayInfo, existing });
    setIsDrawerOpen(true);
  };

  const closeDrawer = () => {
    setIsDrawerOpen(false);
    setTimeout(() => setSelectedCell(null), 300);
  };

  // --- EXPORTAR PARA CSV ---
  const exportToCSV = () => {
    const currentMonthStr = `${currentDate.getFullYear()}-${String(currentDate.getMonth() + 1).padStart(2, '0')}`;
    const pmsThisMonth = pmData.filter(pm => pm.dataStr && pm.dataStr.startsWith(currentMonthStr));
    
    if (pmsThisMonth.length === 0) {
      alert("Não há dados de manutenção neste mês para exportar.");
      return;
    }

    const headers = ['ID', 'Linha/Recurso', 'Data', 'Semana', 'Status', 'Turno', 'Duracao_Horas', 'Atualizado_Por', 'Observacoes'];
    const csvRows = pmsThisMonth.map(pm => {
      return [
        pm.id,
        pm.linha,
        pm.dataStr,
        `W${getWeekNumber(new Date(pm.dataStr))}`,
        pm.status.toUpperCase(),
        pm.turno || 'N/A',
        pm.duracao || '0',
        pm.updatedBy || 'N/A',
        `"${(pm.obs || '').replace(/"/g, '""')}"`
      ].join(',');
    });

    const csvContent = [headers.join(','), ...csvRows].join('\n');
    const blob = new Blob([`\uFEFF${csvContent}`], { type: 'text/csv;charset=utf-8;' }); 
    const url = URL.createObjectURL(blob);
    const link = document.createElement('a');
    link.href = url;
    link.setAttribute('download', `Heineken_PMDay_Alagoinhas_${currentMonthStr}.csv`);
    document.body.appendChild(link);
    link.click();
    document.body.removeChild(link);
  };

  // --- OPERAÇÕES (CRUD) COM FALLBACK PARA MODO LOCAL ---
  const savePM = async (e) => {
    e.preventDefault();

    const formData = new FormData(e.target);
    const status = formData.get('status');
    const obs = formData.get('obs');
    const turno = formData.get('turno');
    const duracao = formData.get('duracao');
    
    const pmId = selectedCell.existing ? selectedCell.existing.id.toString() : `pm_${Date.now()}`;
    
    const pmPayload = {
      id: pmId,
      linha: selectedCell.linha,
      dataStr: selectedCell.dayInfo.dateStr,
      status,
      turno,
      duracao: Number(duracao),
      obs,
      updatedAt: new Date().toISOString(),
      updatedBy: loggedEmail
    };

    if (!user || syncError || !db || !safeAppId) {
      setPmData(prev => {
        const index = prev.findIndex(p => p.id === pmPayload.id);
        if (index >= 0) {
          const newArray = [...prev];
          newArray[index] = pmPayload;
          return newArray;
        }
        return [...prev, pmPayload];
      });
      closeDrawer();
      return;
    }

    try {
      const pmRef = ref(db, `artifacts/${safeAppId}/public/data/${DB_PATH}/${pmId}`);
      await set(pmRef, pmPayload);
      closeDrawer();
    } catch (error) {
      console.error("Erro ao guardar PMDay na nuvem:", error);
      setPmData(prev => {
        const index = prev.findIndex(p => p.id === pmPayload.id);
        if (index >= 0) {
          const newArray = [...prev];
          newArray[index] = pmPayload;
          return newArray;
        }
        return [...prev, pmPayload];
      });
      closeDrawer();
    }
  };

  const removePM = async () => {
    if (!selectedCell.existing) return;
    const targetId = selectedCell.existing.id.toString();

    if (!user || syncError || !db || !safeAppId) {
      setPmData(prev => prev.filter(p => p.id !== targetId));
      closeDrawer();
      return;
    }

    try {
      const pmRef = ref(db, `artifacts/${safeAppId}/public/data/${DB_PATH}/${targetId}`);
      await remove(pmRef);
      closeDrawer();
    } catch (error) {
      console.error("Erro ao remover PMDay na nuvem:", error);
    }
  };

  const prevMonth = () => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() - 1, 1));
  const nextMonth = () => setCurrentDate(new Date(currentDate.getFullYear(), currentDate.getMonth() + 1, 1));

  // --- TELA DE LOGIN / REGISTO ---
  if (!isLoggedIn) {
    return (
      <div className="min-h-screen bg-[#F5F7FA] flex flex-col items-center justify-center p-4 font-sans relative overflow-hidden">
        <div className="absolute top-0 left-0 w-full h-72 bg-gradient-to-b from-[#008248] to-[#005b32] rounded-b-[50%] scale-x-150 shadow-lg"></div>
        
        <div className="relative z-10 w-full max-w-md">
          <div className="bg-white rounded-3xl shadow-2xl overflow-hidden border border-gray-100 transition-all duration-300">
            <div className="p-10 text-center bg-gray-50 border-b border-gray-100 flex flex-col items-center">
              <div className="flex items-center gap-2 mb-3">
                <svg className="w-12 h-12 text-[#FF2B2B] drop-shadow-sm" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M12 2.5l3.09 6.26 6.91 1.01-5 4.87 1.18 6.88L12 18.27l-6.18 3.25 1.18-6.88-5-4.87 6.91-1.01L12 2.5z" />
                </svg>
                <h1 className="text-4xl font-black text-[#008248] tracking-tight" style={{ fontFamily: 'Georgia, serif' }}>
                  Heineken<span className="text-[14px] font-sans align-top ml-0.5 opacity-80">®</span>
                </h1>
              </div>
              <h2 className="text-sm font-bold text-gray-500 tracking-widest uppercase">PMDay Portal</h2>
              <p className="text-xs text-gray-400 mt-2">
                {isRegistering ? 'Crie a sua conta de acesso' : 'Acesso restrito a colaboradores'}
              </p>
            </div>
            
            <form onSubmit={handleLoginSubmit} className="p-8 space-y-5">
              {isRegistering && (
                <div className="space-y-2 animate-in fade-in slide-in-from-top-4 duration-300">
                  <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Nome Completo</label>
                  <input 
                    required 
                    type="text" 
                    value={registerName}
                    onChange={(e) => setRegisterName(e.target.value)}
                    placeholder="João Silva" 
                    className="w-full p-4 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none text-sm font-semibold transition-all text-gray-700" 
                  />
                </div>
              )}

              <div className="space-y-2">
                <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">E-mail Corporativo</label>
                <input 
                  required 
                  type="email" 
                  value={loginEmail}
                  onChange={(e) => setLoginEmail(e.target.value)}
                  placeholder="nome.sobrenome@heineken.com" 
                  className="w-full p-4 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none text-sm font-semibold transition-all text-gray-700" 
                />
              </div>
              
              <div className="space-y-2">
                <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Palavra-passe</label>
                <input required type="password" placeholder="••••••••" className="w-full p-4 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none text-sm font-semibold transition-all text-gray-700" />
              </div>

              {isRegistering && (
                <div className="space-y-2 animate-in fade-in slide-in-from-top-4 duration-300">
                  <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Confirmar Palavra-passe</label>
                  <input required type="password" placeholder="••••••••" className="w-full p-4 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none text-sm font-semibold transition-all text-gray-700" />
                </div>
              )}

              <button type="submit" className="w-full py-4 bg-[#008248] text-white rounded-xl font-bold text-sm shadow-md shadow-green-200 hover:bg-[#005b32] hover:shadow-lg transition-all flex justify-center items-center gap-2 mt-2">
                {isRegistering ? 'Criar Conta' : 'Aceder ao Sistema'}
              </button>

              <div className="text-center pt-2 border-t border-gray-100 mt-6">
                <button 
                  type="button" 
                  onClick={() => setIsRegistering(!isRegistering)} 
                  className="text-xs font-bold text-gray-500 hover:text-[#008248] transition-colors"
                >
                  {isRegistering ? 'Já tem uma conta? Iniciar Sessão' : 'Não tem conta? Solicite ou Crie uma Conta'}
                </button>
              </div>
            </form>
          </div>
          
          <div className="mt-8 text-center text-sm font-medium text-gray-500 flex flex-col items-center gap-2">
            <span>© {new Date().getFullYear()} Heineken OEE Management</span>
            <span className="flex items-center gap-1.5 bg-white px-4 py-2 rounded-full shadow-sm border border-gray-200">
              Desenvolvido por 
              <a href="https://www.linkedin.com/in/7icaaro" target="_blank" rel="noopener noreferrer" className="text-[#008248] font-black hover:underline flex items-center gap-1.5 transition-colors">
                Icaro
                <svg className="w-4 h-4 text-[#0a66c2]" fill="currentColor" viewBox="0 0 24 24"><path d="M19 0h-14c-2.761 0-5 2.239-5 5v14c0 2.761 2.239 5 5 5h14c2.762 0 5-2.239 5-5v-14c0-2.761-2.238-5-5-5zm-11 19h-3v-11h3v11zm-1.5-12.268c-.966 0-1.75-.79-1.75-1.764s.784-1.764 1.75-1.764 1.75.79 1.75 1.764-.783 1.764-1.75 1.764zm13.5 12.268h-3v-5.604c0-3.368-4-3.113-4 0v5.604h-3v-11h3v1.765c1.396-2.586 7-2.777 7 2.476v6.759z"/></svg>
              </a>
            </span>
          </div>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-[#F5F7FA] flex flex-col font-sans overflow-hidden">
      {/* Navbar Profissional */}
      <header className="bg-white border-b border-gray-200 shadow-sm z-40 relative">
        <div className="max-w-[1800px] mx-auto px-6 h-16 flex items-center justify-between">
          <div className="flex items-center gap-4">
            <div className="flex items-center gap-2 mr-2">
              <svg className="w-10 h-10 text-[#FF2B2B] drop-shadow-sm transform hover:scale-105 transition-transform" fill="currentColor" viewBox="0 0 24 24">
                <path d="M12 2.5l3.09 6.26 6.91 1.01-5 4.87 1.18 6.88L12 18.27l-6.18 3.25 1.18-6.88-5-4.87 6.91-1.01L12 2.5z" />
              </svg>
              <h1 className="text-3xl font-black text-[#008248] tracking-tight" style={{ fontFamily: 'Georgia, serif' }}>
                Heineken<span className="text-[12px] font-sans align-top ml-0.5 opacity-80">®</span>
              </h1>
            </div>
            
            <div className="h-8 w-px bg-gray-200 mx-2 hidden md:block"></div>
            
            <div>
              <h2 className="text-lg font-black text-gray-800 leading-none tracking-tight">PMDay</h2>
              <p className="text-[10px] font-bold text-gray-400 tracking-widest uppercase mt-1">Alagoinhas • OEE Management</p>
            </div>
            
            <div className="ml-4 hidden md:flex items-center gap-2 px-3 py-1.5 rounded-full bg-gray-50 border border-gray-200">
              {isSyncing ? (
                <><Loader2 size={12} className="text-blue-500 animate-spin" /><span className="text-[10px] font-bold text-gray-500">A Iniciar...</span></>
              ) : syncError ? (
                <><CloudOff size={12} className="text-yellow-600" /><span className="text-[10px] font-bold text-yellow-600">Modo Local (Firebase Offline)</span></>
              ) : (
                <><Cloud size={12} className="text-[#008248]" /><span className="text-[10px] font-bold text-[#008248]">Nuvem Conectada</span></>
              )}
            </div>
          </div>

          <div className="flex items-center gap-6">
            <div className="hidden lg:flex items-center gap-2 px-4 py-1.5 bg-gray-900 text-green-400 rounded-lg font-mono text-sm font-bold shadow-inner mr-2">
              <Clock size={14} />
              {currentTime.toLocaleTimeString('pt-PT', { hour12: false })}
            </div>

            <div className="hidden lg:flex flex-col items-end border-r border-gray-200 pr-5">
              <span className="text-xs font-bold text-gray-800">{loggedEmail}</span>
              <div className="flex items-center gap-1 mt-0.5">
                {isUserAdmin ? (
                  <><ShieldCheck size={12} className="text-[#008248]" /><span className="text-[10px] font-black uppercase text-[#008248] tracking-widest">Admin</span></>
                ) : (
                  <><ShieldAlert size={12} className="text-gray-400" /><span className="text-[10px] font-black uppercase text-gray-400 tracking-widest">Visualizador</span></>
                )}
              </div>
            </div>
            
            <div className="flex items-center gap-3">
              <div className="w-8 h-8 rounded-full bg-blue-50 flex items-center justify-center text-[#2563EB] border border-blue-100 cursor-pointer hover:bg-blue-100 transition-colors">
                <Settings size={16} />
              </div>
            </div>
          </div>
        </div>
      </header>

      {/* Main Workspace */}
      <main className="flex-1 flex flex-col p-6 max-w-[1800px] mx-auto w-full gap-6">
        
        {/* Painel de Controle */}
        <div className="bg-white p-4 rounded-2xl border border-gray-200 shadow-sm flex flex-col md:flex-row justify-between items-center gap-4">
          <div className="flex items-center gap-3 w-full md:w-auto overflow-x-auto pb-2 md:pb-0">
            <div className="flex items-center bg-gray-50 border border-gray-200 rounded-xl p-1 shrink-0">
              <button onClick={prevMonth} className="p-2 hover:bg-white rounded-lg hover:shadow-sm transition-all text-gray-500 hover:text-[#008248]"><ChevronLeft size={18}/></button>
              <div className="px-6 flex items-center min-w-[180px] justify-center">
                <Calendar className="w-4 h-4 mr-2 text-[#008248]" />
                <span className="text-sm font-bold text-gray-800 capitalize">
                  {currentDate.toLocaleDateString('pt-PT', { month: 'long', year: 'numeric' })}
                </span>
              </div>
              <button onClick={nextMonth} className="p-2 hover:bg-white rounded-lg hover:shadow-sm transition-all text-gray-500 hover:text-[#008248]"><ChevronRight size={18}/></button>
            </div>
            
            <div className="h-6 w-px bg-gray-200 mx-2 hidden md:block"></div>
            
            <div className="flex gap-4 items-center shrink-0">
              <div className="flex items-center gap-2">
                <div className="w-3 h-3 rounded-full bg-[#2563EB]"></div>
                <span className="text-xs font-semibold text-gray-600">Planeado</span>
              </div>
              <div className="flex items-center gap-2">
                <div className="w-3 h-3 rounded-full bg-[#008248]"></div>
                <span className="text-xs font-semibold text-gray-600">Realizado</span>
              </div>
            </div>
          </div>

          <div className="flex gap-3 w-full md:w-auto">
             <div className="relative flex-1 md:flex-none">
               <Search className="w-4 h-4 absolute left-3 top-1/2 -translate-y-1/2 text-gray-400" />
               <input 
                 type="text" 
                 placeholder="Filtrar linha/recurso..." 
                 value={searchTerm}
                 onChange={(e) => setSearchTerm(e.target.value)}
                 className="w-full pl-9 pr-4 py-2.5 bg-gray-50 border border-gray-200 rounded-xl text-sm focus:outline-none focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] transition-all" 
               />
             </div>
             <button 
               onClick={exportToCSV}
               className="flex items-center justify-center gap-2 px-5 py-2.5 bg-white border border-gray-200 rounded-xl text-sm font-bold text-[#008248] hover:bg-green-50 hover:border-green-200 transition-all shadow-sm whitespace-nowrap"
             >
               <FileDown size={18} /> CSV
             </button>
          </div>
        </div>

        {/* Dashboard Analítico - Agenda Completa */}
        <div className="grid grid-cols-2 md:grid-cols-5 gap-4">
          <div className="bg-white p-5 rounded-2xl border border-gray-200 shadow-sm flex items-center gap-4 hover:shadow-md transition-shadow">
            <div className="w-12 h-12 rounded-full bg-blue-50 flex items-center justify-center text-[#2563EB]">
              <BarChart3 size={24} />
            </div>
            <div>
              <p className="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Total (Mês)</p>
              <p className="text-2xl font-black text-gray-800">{metrics.total}</p>
            </div>
          </div>
          
          <div className="bg-white p-5 rounded-2xl border border-gray-200 shadow-sm flex items-center gap-4 hover:shadow-md transition-shadow">
            <div className="w-12 h-12 rounded-full bg-blue-50/50 flex items-center justify-center text-[#2563EB]">
              <Clock size={24} />
            </div>
            <div>
              <p className="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Programados</p>
              <p className="text-2xl font-black text-gray-800">{metrics.programados}</p>
            </div>
          </div>
          
          <div className="bg-white p-5 rounded-2xl border border-gray-200 shadow-sm flex items-center gap-4 hover:shadow-md transition-shadow">
            <div className="w-12 h-12 rounded-full bg-green-50 flex items-center justify-center text-[#008248]">
              <CheckCircle2 size={24} />
            </div>
            <div>
              <p className="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Realizados</p>
              <p className="text-2xl font-black text-[#008248]">{metrics.realizados}</p>
            </div>
          </div>
          
          <div className="bg-white p-5 rounded-2xl border border-gray-200 shadow-sm flex items-center gap-4 hover:shadow-md transition-shadow relative overflow-hidden">
            <div className={`absolute bottom-0 left-0 h-1 transition-all duration-1000 ${metrics.taxa === 100 ? 'bg-[#008248]' : 'bg-blue-500'}`} style={{ width: `${metrics.taxa}%` }}></div>
            <div className="w-12 h-12 rounded-full bg-purple-50 flex items-center justify-center text-purple-600">
              <TrendingUp size={24} />
            </div>
            <div>
              <p className="text-[10px] font-bold text-gray-400 uppercase tracking-widest">Aderência</p>
              <p className="text-2xl font-black text-gray-800">{metrics.taxa}%</p>
            </div>
          </div>

          <div className="col-span-2 md:col-span-1 bg-[#008248] p-5 rounded-2xl shadow-sm flex items-center gap-4 hover:shadow-md transition-shadow text-white relative overflow-hidden">
            <div className="absolute right-0 top-0 w-32 h-32 bg-white/10 rounded-full blur-2xl -mr-10 -mt-10"></div>
            <div className="w-12 h-12 rounded-full bg-white/20 flex items-center justify-center backdrop-blur-sm z-10">
              <Timer size={24} />
            </div>
            <div className="z-10">
              <p className="text-[10px] font-bold text-green-100 uppercase tracking-widest">Esforço (Horas)</p>
              <p className="text-2xl font-black">{metrics.horasTotais}h</p>
            </div>
          </div>
        </div>

        {/* Data Grid Profissional */}
        <div className="bg-white rounded-2xl border border-gray-200 shadow-sm overflow-hidden flex-1 relative">
          
          {isSyncing && !syncError && pmData.length === 0 && (
             <div className="absolute inset-0 bg-white/80 backdrop-blur-sm z-30 flex flex-col items-center justify-center">
                <Loader2 size={32} className="text-[#008248] animate-spin mb-4" />
                <p className="text-sm font-bold text-gray-600 uppercase tracking-widest">A iniciar ambiente...</p>
             </div>
          )}

          <div className="overflow-x-auto h-full">
            <table className="w-full border-collapse text-sm">
              <thead>
                <tr>
                  <th className="sticky left-0 z-30 bg-white border-b border-r border-gray-200 min-w-[180px] p-4 text-left shadow-[4px_0_12px_rgba(0,0,0,0.03)]">
                    <span className="text-xs font-bold text-gray-400 uppercase tracking-wider">Recurso / Linha</span>
                  </th>
                  {monthData.map((week, idx) => (
                    <th key={`w-${week.weekNumber}`} colSpan={week.days.length + 1} className="bg-gray-50/80 border-b border-r border-gray-200 p-2 text-center relative overflow-hidden">
                      <div className="absolute inset-0 bg-gradient-to-r from-transparent via-white/40 to-transparent"></div>
                      <span className="text-[10px] font-black text-gray-500 uppercase tracking-widest relative z-10">
                        WEEK {week.weekNumber}
                      </span>
                    </th>
                  ))}
                </tr>
                <tr>
                  <th className="sticky left-0 z-30 bg-white border-b border-r border-gray-200 shadow-[4px_0_12px_rgba(0,0,0,0.03)]"></th>
                  {monthData.map(week => (
                    <React.Fragment key={`header-week-${week.weekNumber}`}>
                      {week.days.map((day, idx) => (
                        <th key={`d-${day.dateStr}`} className="bg-white border-b border-r border-gray-100 p-2 min-w-[56px] text-center">
                          <div className="flex flex-col items-center">
                            <span className="text-[10px] font-bold text-gray-400 uppercase">{day.weekDayName}</span>
                            <span className={`text-sm font-black mt-0.5 ${day.dateStr === new Date().toISOString().split('T')[0] ? 'text-[#008248] bg-green-50 w-7 h-7 rounded-full flex items-center justify-center' : 'text-gray-700'}`}>
                              {day.dayNumber}
                            </span>
                          </div>
                        </th>
                      ))}
                      <th className="bg-gray-50/80 border-b border-r-2 border-r-gray-200 border-b-gray-200 p-2 min-w-[60px] text-center shadow-[inset_2px_0_4px_rgba(0,0,0,0.01)]">
                        <span className="text-[9px] font-bold text-[#008248] uppercase tracking-widest">Status</span>
                      </th>
                    </React.Fragment>
                  ))}
                </tr>
              </thead>
              <tbody>
                {filteredLinhas.length === 0 && (
                  <tr>
                    <td colSpan={100} className="p-10 text-center text-gray-400 font-medium">
                      Nenhuma linha encontrada para o filtro "{searchTerm}"
                    </td>
                  </tr>
                )}
                {filteredLinhas.map((linha) => (
                  <tr key={linha} className="group hover:bg-gray-50/50 transition-colors">
                    <td className="sticky left-0 z-20 bg-white group-hover:bg-gray-50/50 border-b border-r border-gray-100 p-4 shadow-[4px_0_12px_rgba(0,0,0,0.02)] transition-colors">
                      <div className="flex items-center gap-3">
                        <Activity className="w-4 h-4 text-gray-300" />
                        <span className="font-semibold text-gray-700 text-sm whitespace-nowrap">{linha}</span>
                      </div>
                    </td>
                    
                    {monthData.map(week => {
                      const hasPM = week.days.some(day => getCellData(linha, day.dateStr));
                      return (
                        <React.Fragment key={`body-week-${week.weekNumber}`}>
                          {week.days.map(day => {
                            const pm = getCellData(linha, day.dateStr);
                            return (
                              <td 
                                key={`${linha}-${day.dateStr}`} 
                                onClick={() => handleCellClick(linha, day)}
                                className={`border-b border-r border-gray-50 p-1.5 relative ${isUserAdmin ? 'cursor-pointer hover:bg-gray-100' : 'cursor-not-allowed'} transition-colors group/cell`}
                              >
                                <div className="w-full h-10 rounded-lg flex items-center justify-center relative">
                                  {pm ? (
                                    <>
                                      <div className={`absolute inset-1 rounded-md shadow-sm border flex items-center justify-center transition-all duration-300 ${isUserAdmin ? 'hover:scale-[1.05]' : ''} ${
                                        pm.status === 'programado' ? 'bg-[#2563EB] border-blue-600' : 'bg-[#008248] border-green-700'
                                      }`}>
                                        {pm.status === 'realizado' ? (
                                          <CheckCircle2 size={14} className="text-white" />
                                        ) : (
                                          <Clock size={14} className="text-white opacity-90 animate-pulse" />
                                        )}
                                      </div>
                                      
                                      <div className="absolute bottom-full left-1/2 -translate-x-1/2 mb-2 hidden group-hover/cell:block z-50 w-56 bg-gray-900 text-white text-[10px] p-3 rounded-lg shadow-xl border border-gray-700 pointer-events-none">
                                        <div className="flex justify-between items-start mb-2">
                                          <p className="font-bold uppercase text-xs">{pm.status === 'programado' ? '📅 Programado' : '✅ Realizado'}</p>
                                          <span className="bg-gray-800 px-2 py-1 rounded text-[9px] text-gray-300">{pm.duracao}h</span>
                                        </div>
                                        <p className="opacity-80 border-b border-gray-700 pb-2 mb-2">Turno: <span className="font-bold text-white">{pm.turno || 'Não definido'}</span></p>
                                        <div className="text-gray-300 italic line-clamp-3">
                                          {pm.obs || 'Sem observações técnicas.'}
                                        </div>
                                        {pm.updatedBy && (
                                          <div className="mt-2 pt-2 border-t border-gray-700 text-gray-400 text-[9px]">
                                            Modificado por: <span className="text-[#008248] font-bold">{pm.updatedBy}</span>
                                          </div>
                                        )}
                                      </div>
                                    </>
                                  ) : (
                                    <div className={`w-full h-full border-2 border-dashed border-transparent ${isUserAdmin ? 'group-hover/cell:border-gray-200' : ''} rounded-md transition-all`}></div>
                                  )}
                                </div>
                              </td>
                            );
                          })}
                          <td className="border-b border-r-2 border-r-gray-200 border-b-gray-50 p-1.5 text-center bg-gray-50/30">
                            <div className="flex items-center justify-center h-10">
                              {hasPM ? (
                                <div className="w-7 h-7 rounded bg-green-100 border border-green-200 flex items-center justify-center text-[#008248]" title="PM Planejado ou Realizado nesta semana">
                                  <CheckCircle2 size={16} />
                                </div>
                              ) : (
                                <div className="w-7 h-7 rounded bg-gray-100/50 border border-gray-200 flex items-center justify-center text-gray-400" title="Sem PM nesta semana">
                                  <span className="text-sm font-bold">-</span>
                                </div>
                              )}
                            </div>
                          </td>
                        </React.Fragment>
                      );
                    })}
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
        </div>
      </main>

      <div className={`fixed inset-y-0 right-0 w-full md:w-[450px] bg-white shadow-2xl z-50 transform transition-transform duration-300 ease-in-out border-l border-gray-200 flex flex-col ${isDrawerOpen ? 'translate-x-0' : 'translate-x-full'}`}>
        {selectedCell && (
          <>
            <div className="p-6 border-b border-gray-100 bg-gray-50 flex justify-between items-start">
              <div>
                <div className="flex items-center gap-2 text-xs font-bold text-[#008248] uppercase tracking-widest mb-2">
                  <Calendar size={14} /> Lançamento de Ordem
                </div>
                <h2 className="text-2xl font-black text-gray-800">{selectedCell.linha}</h2>
                <p className="text-sm text-gray-500 mt-1 font-medium">
                  {selectedCell.dayInfo.dayNumber} de {currentDate.toLocaleDateString('pt-PT', { month: 'long' })} ({selectedCell.dayInfo.weekDayName})
                </p>
              </div>
              <button onClick={closeDrawer} className="p-2 bg-white rounded-full border border-gray-200 hover:bg-gray-100 transition-colors text-gray-500">
                <X size={18} />
              </button>
            </div>

            <form onSubmit={savePM} className="p-6 flex-1 overflow-y-auto flex flex-col gap-6">
              <div className="space-y-3">
                <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Status Operacional</label>
                <div className="grid grid-cols-2 gap-3">
                  <label className="cursor-pointer">
                    <input type="radio" name="status" value="programado" className="peer sr-only" defaultChecked={!selectedCell.existing || selectedCell.existing.status === 'programado'} />
                    <div className="p-4 rounded-xl border-2 border-gray-100 peer-checked:border-[#2563EB] peer-checked:bg-blue-50 transition-all flex flex-col items-center gap-2 text-gray-500 peer-checked:text-[#2563EB]">
                      <Clock size={24} />
                      <span className="text-sm font-bold">Programado</span>
                    </div>
                  </label>
                  <label className="cursor-pointer">
                    <input type="radio" name="status" value="realizado" className="peer sr-only" defaultChecked={selectedCell.existing?.status === 'realizado'} />
                    <div className="p-4 rounded-xl border-2 border-gray-100 peer-checked:border-[#008248] peer-checked:bg-green-50 transition-all flex flex-col items-center gap-2 text-gray-500 peer-checked:text-[#008248]">
                      <CheckCircle2 size={24} />
                      <span className="text-sm font-bold">Realizado</span>
                    </div>
                  </label>
                </div>
              </div>

              <div className="grid grid-cols-2 gap-4">
                <div className="space-y-2">
                  <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Turno</label>
                  <select 
                    name="turno" 
                    defaultValue={selectedCell.existing?.turno || 'Manhã'}
                    className="w-full p-3.5 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none text-sm font-semibold transition-all text-gray-700"
                  >
                    <option value="Manhã">Manhã (A)</option>
                    <option value="Tarde">Tarde (B)</option>
                    <option value="Noite">Noite (C)</option>
                  </select>
                </div>
                <div className="space-y-2">
                  <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Duração (Horas)</label>
                  <div className="relative">
                    <input 
                      type="number" 
                      name="duracao" 
                      min="1" 
                      max="24"
                      defaultValue={selectedCell.existing?.duracao || 8}
                      className="w-full pl-4 pr-10 py-3.5 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none text-sm font-semibold transition-all text-gray-700"
                    />
                    <span className="absolute right-4 top-1/2 -translate-y-1/2 text-gray-400 text-sm font-bold">h</span>
                  </div>
                </div>
              </div>

              <div className="space-y-3">
                <label className="text-xs font-bold text-gray-500 uppercase tracking-wider">Escopo Técnico / Observações</label>
                <textarea 
                  name="obs"
                  defaultValue={selectedCell.existing?.obs || ''}
                  placeholder="Ex: Substituição de rolamentos, limpeza profunda CIP..."
                  className="w-full p-4 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#008248]/20 focus:border-[#008248] outline-none h-32 resize-none text-sm transition-all text-gray-700"
                ></textarea>
              </div>

              <div className="mt-auto pt-6 flex gap-3 border-t border-gray-100">
                {selectedCell.existing && (
                  <button 
                    type="button"
                    onClick={removePM}
                    className="px-6 py-3 text-sm font-bold text-red-600 bg-red-50 hover:bg-red-100 rounded-xl transition-colors"
                  >
                    Excluir
                  </button>
                )}
                <button 
                  type="submit"
                  className="flex-1 py-3 bg-[#008248] text-white rounded-xl font-bold text-sm shadow-md shadow-green-200 hover:bg-[#005b32] hover:shadow-lg transition-all flex justify-center items-center gap-2"
                >
                  <Cloud size={18} /> Salvar PMDay
                </button>
              </div>
            </form>
          </>
        )}
      </div>
      
      {isDrawerOpen && (
        <div className="fixed inset-0 bg-gray-900/20 backdrop-blur-[2px] z-40 transition-opacity" onClick={closeDrawer}></div>
      )}
      
      <footer className="bg-white border-t border-gray-200 py-3 mt-auto text-center z-30">
        <span className="text-xs font-semibold text-gray-400 flex items-center justify-center gap-1.5">
          PMDay Portal • Desenvolvido por 
          <a href="https://www.linkedin.com/in/7icaaro" target="_blank" rel="noopener noreferrer" className="text-[#008248] hover:underline flex items-center gap-1">
            Icaro
            <svg className="w-3 h-3 text-[#0a66c2]" fill="currentColor" viewBox="0 0 24 24"><path d="M19 0h-14c-2.761 0-5 2.239-5 5v14c0 2.761 2.239 5 5 5h14c2.762 0 5-2.239 5-5v-14c0-2.761-2.238-5-5-5zm-11 19h-3v-11h3v11zm-1.5-12.268c-.966 0-1.75-.79-1.75-1.764s.784-1.764 1.75-1.764 1.75.79 1.75 1.764-.783 1.764-1.75 1.764zm13.5 12.268h-3v-5.604c0-3.368-4-3.113-4 0v5.604h-3v-11h3v1.765c1.396-2.586 7-2.777 7 2.476v6.759z"/></svg>
          </a>
        </span>
      </footer>
    </div>
  );
};

export default App;
