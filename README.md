import React, { useState, useEffect, useRef, Suspense } from 'react';
import { Canvas } from '@react-three/fiber';
import { Sphere, MeshDistortMaterial, OrbitControls } from '@react-three/drei';
import { motion, AnimatePresence } from 'framer-motion';
import { Heart, Plus, User, ShieldCheck, X, Trash2, LogIn, LogOut, Image as ImageIcon, Video } from 'lucide-react';

// --- КОНСТАНТЫ ---
const SECRET_CODE = "88888888$";

export default function JukeApp() {
  const [posts, setPosts] = useState([]);
  const [user, setUser] = useState(null);
  const [isAdmin, setIsAdmin] = useState(false);
  const [showAdminLogin, setShowAdminLogin] = useState(false);
  const [inputCode, setInputCode] = useState('');
  const [selectedFile, setSelectedFile] = useState(null);
  const fileInputRef = useRef(null);

  // --- ИНИЦИАЛИЗАЦИЯ (LocalStorage) ---
  useEffect(() => {
    const savedPosts = localStorage.getItem('juke_posts');
    const savedUser = localStorage.getItem('juke_user');
    if (savedPosts) setPosts(JSON.parse(savedPosts));
    if (savedUser) setUser(JSON.parse(savedUser));
  }, []);

  useEffect(() => {
    localStorage.setItem('juke_posts', JSON.stringify(posts));
  }, [posts]);

  // --- ЛОГИКА АККАУНТА ---
  const handleGoogleSignIn = () => {
    // Имитация входа через Google для Web-версии
    const mockUser = {
      id: 'user_' + Math.random().toString(36).substr(2, 9),
      name: "Google User",
      photo: `https://api.dicebear.com/7.x/pixel-art/svg?seed=${Math.random()}`
    };
    setUser(mockUser);
    localStorage.setItem('juke_user', JSON.stringify(mockUser));
  };

  const handleLogout = () => {
    setUser(null);
    setIsAdmin(false);
    localStorage.removeItem('juke_user');
  };

  // --- АДМИН ПАНЕЛЬ ---
  const handleAdminAuth = () => {
    if (inputCode === SECRET_CODE) {
      setIsAdmin(true);
      setShowAdminLogin(false);
      setInputCode('');
    } else {
      alert("Неверный код доступа");
    }
  };

  // --- РАБОТА С КОНТЕНТОМ ---
  const handleFileSelect = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    const reader = new FileReader();
    reader.onloadend = () => {
      setSelectedFile({
        url: reader.result,
        type: file.type.startsWith('video') ? 'video' : 'image'
      });
    };
    reader.readAsDataURL(file);
  };

  const submitPost = (e) => {
    e.preventDefault();
    const text = e.target.content.value;
    if (!text && !selectedFile) return;

    const newPost = {
      id: Date.now(),
      author: user?.name || "Guest",
      authorImg: user?.photo || "",
      content: text,
      media: selectedFile?.url,
      mediaType: selectedFile?.type,
      likes: 0,
      ownerId: user?.id || 'admin'
    };

    setPosts([newPost, ...posts]);
    setSelectedFile(null);
    e.target.reset();
  };

  const deletePost = (id) => {
    setPosts(posts.filter(p => p.id !== id));
  };

  return (
    <div className="min-h-screen bg-[#020202] text-white font-sans selection:bg-cyan-500">
      <BackgroundCanvas />

      {/* ШАПКА */}
      <nav className="fixed top-0 w-full z-50 backdrop-blur-md border-b border-white/5 px-6 py-4 flex justify-between items-center">
        <h1 className="text-2xl font-black italic bg-gradient-to-r from-cyan-400 to-purple-500 bg-clip-text text-transparent tracking-tighter">
          jUkE
        </h1>
        
        <div className="flex items-center gap-4">
          {user ? (
            <div className="flex items-center gap-3 bg-white/5 py-1 px-1 pr-3 rounded-full border border-white/10">
              <img src={user.photo} className="w-8 h-8 rounded-full border border-cyan-500/30" alt="me" />
              <button onClick={handleLogout} className="text-white/40 hover:text-white"><LogOut size={18}/></button>
            </div>
          ) : (
            <button onClick={handleGoogleSignIn} className="bg-white text-black px-4 py-1.5 rounded-full font-bold text-sm flex items-center gap-2">
              <LogIn size={16}/> Sign In
            </button>
          )}
          
          <button onClick={() => isAdmin ? setIsAdmin(false) : setShowAdminLogin(true)}>
            <ShieldCheck size={24} className={isAdmin ? "text-cyan-400" : "text-white/20"} />
          </button>
        </div>
      </nav>

      {/* ЛЕНТА */}
      <main className="max-w-lg mx-auto pt-24 pb-12 px-4 space-y-8">
        
        {/* ФОРМА ПОСТА (Видна если залогинен или админ) */}
        {(user || isAdmin) && (
          <motion.div initial={{ scale: 0.95, opacity: 0 }} animate={{ scale: 1, opacity: 1 }} className="bg-white/5 border border-white/10 p-5 rounded-[2rem] backdrop-blur-2xl">
            <form onSubmit={submitPost} className="space-y-4">
              <textarea 
                name="content" 
                placeholder="Что нового?" 
                className="w-full bg-transparent border-none outline-none text-lg resize-none placeholder:text-white/20"
              />
              
              {selectedFile && (
                <div className="relative rounded-2xl overflow-hidden border border-white/10">
                  {selectedFile.type === 'video' ? 
                    <video src={selectedFile.url} className="w-full" autoPlay muted loop /> : 
                    <img src={selectedFile.url} className="w-full" alt="upload" />
                  }
                  <button onClick={() => setSelectedFile(null)} className="absolute top-2 right-2 bg-black/60 p-2 rounded-full"><X size={16}/></button>
                </div>
              )}

              <div className="flex justify-between items-center border-t border-white/5 pt-4">
                <div className="flex gap-4 text-white/40">
                  <input type="file" hidden ref={fileInputRef} onChange={handleFileSelect} accept="image/*,video/*" />
                  <button type="button" onClick={() => fileInputRef.current.click()} className="hover:text-cyan-400 transition"><ImageIcon size={22}/></button>
                  <button type="button" onClick={() => fileInputRef.current.click()} className="hover:text-purple-400 transition"><Video size={22}/></button>
                </div>
                <button type="submit" className="bg-gradient-to-r from-cyan-500 to-blue-600 px-6 py-2 rounded-xl font-bold hover:shadow-[0_0_20px_rgba(6,182,212,0.4)] transition">
                  Post
                </button>
              </div>
            </form>
          </motion.div>
        )}

        {/* СПИСОК ПОСТОВ */}
        <div className="space-y-6">
          {posts.map(post => (
            <Post key={post.id} post={post} isAdmin={isAdmin} currentUserId={user?.id} onDelete={() => deletePost(post.id)} />
          ))}
        </div>
      </main>

      {/* МОДАЛКА АДМИНА */}
      <AnimatePresence>
        {showAdminLogin && (
          <motion.div initial={{ opacity: 0 }} animate={{ opacity: 1 }} exit={{ opacity: 0 }} className="fixed inset-0 z-[100] flex items-center justify-center bg-black/90 backdrop-blur-sm">
            <div className="bg-[#111] p-8 rounded-[2.5rem] border border-white/10 w-full max-w-[300px]">
              <h3 className="text-center font-bold mb-6">Admin Key</h3>
              <input 
                type="password" 
                value={inputCode}
                onChange={(e) => setInputCode(e.target.value)}
                className="w-full bg-white/5 border border-white/10 rounded-xl py-3 text-center text-xl tracking-widest outline-none focus:border-cyan-500"
              />
              <button onClick={handleAdminAuth} className="w-full bg-white text-black py-3 rounded-xl font-bold mt-4">Unlock</button>
              <button onClick={() => setShowAdminLogin(false)} className="w-full text-white/30 mt-2 text-sm">Cancel</button>
            </div>
          </motion.div>
        )}
      </AnimatePresence>
    </div>
  );
}

// --- КОМПОНЕНТЫ ---

function Post({ post, isAdmin, currentUserId, onDelete }) {
  const [liked, setLiked] = useState(false);
  const canDelete = isAdmin || (currentUserId && post.ownerId === currentUserId);

  return (
    <motion.div layout className="bg-white/5 border border-white/10 rounded-[2.5rem] overflow-hidden">
      <div className="p-4 flex justify-between items-center">
        <div className="flex items-center gap-3">
          {post.authorImg ? <img src={post.authorImg} className="w-8 h-8 rounded-full" /> : <div className="w-8 h-8 rounded-full bg-gray-800" />}
          <span className="font-bold text-sm">{post.author}</span>
        </div>
        {canDelete && <button onClick={onDelete} className="text-white/20 hover:text-red-500 transition"><Trash2 size={18}/></button>}
      </div>
      
      {post.media && (
        <div className="bg-black/20">
          {post.mediaType === 'video' ? 
            <video src={post.media} controls className="w-full max-h-[500px] object-contain" /> : 
            <img src={post.media} className="w-full max-h-[500px] object-contain" alt="content" />
          }
        </div>
      )}

      <div className="p-6">
        <button onClick={() => setLiked(!liked)} className={`mb-3 transition-transform active:scale-150 ${liked ? 'text-red-500' : 'text-white/20'}`}>
          <Heart fill={liked ? "currentColor" : "none"} size={26} />
        </button>
        <p className="text-white/70 text-sm leading-relaxed">
          <span className="font-bold text-white mr-2">{post.author}</span>{post.content}
        </p>
      </div>
    </motion.div>
  );
}

function BackgroundCanvas() {
  return (
    <div className="fixed inset-0 -z-10 opacity-40">
      <Suspense fallback={null}>
        <Canvas>
          <ambientLight intensity={0.5} />
          <Sphere args={[1, 100, 200]} scale={2.2}>
            <MeshDistortMaterial color="#1e1e3f" speed={1.5} distort={0.4} />
          </Sphere>
          <OrbitControls enableZoom={false} />
        </Canvas>
      </Suspense>
    </div>
  );
}
