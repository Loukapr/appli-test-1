# appli-test-1
git init
git remote add origin https://github.com/ton-utilisateur/memoire-vivante.git
git add .
git commit -m "Première version de l'app Mémoire Vivante"
git push -u origin master
git push -u origin main
git add .
// Structure Next.js avec App Router, Firebase Auth et enregistrement audio

// === 1. FICHIERS ===
// 📁 /app
// ┣ 📄 layout.tsx
// ┣ 📄 page.tsx (accueil)
// ┣ 📁 dashboard/page.tsx (zone utilisateur)
// ┣ 📁 record/page.tsx (enregistrement)
// 📁 /components
// ┣ 📄 AudioRecorder.tsx
// 📄 firebase.ts (config)
// 📁 /lib
// ┣ 📄 auth.ts (auth utils)
// 📄 .env.local (non commité)

// === 2. Code principal simplifié ===

// firebase.ts
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);

// lib/auth.ts
import { GoogleAuthProvider, signInWithPopup, signOut } from 'firebase/auth';
import { auth } from '../firebase';

export const login = async () => {
  const provider = new GoogleAuthProvider();
  await signInWithPopup(auth, provider);
};

export const logout = async () => {
  await signOut(auth);
};

// components/AudioRecorder.tsx
'use client';
import { useState, useRef } from 'react';

export default function AudioRecorder() {
  const [recording, setRecording] = useState(false);
  const [audioURL, setAudioURL] = useState('');
  const mediaRecorderRef = useRef<MediaRecorder | null>(null);
  const audioChunks = useRef<Blob[]>([]);

  const startRecording = async () => {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    mediaRecorderRef.current = new MediaRecorder(stream);
    mediaRecorderRef.current.ondataavailable = e => audioChunks.current.push(e.data);
    mediaRecorderRef.current.onstop = () => {
      const blob = new Blob(audioChunks.current, { type: 'audio/webm' });
      setAudioURL(URL.createObjectURL(blob));
      audioChunks.current = [];
    };
    mediaRecorderRef.current.start();
    setRecording(true);
  };

  const stopRecording = () => {
    mediaRecorderRef.current?.stop();
    setRecording(false);
  };

  return (
    <div className="p-4 border rounded-xl">
      <button onClick={recording ? stopRecording : startRecording} className="px-4 py-2 bg-blue-500 text-white rounded">
        {recording ? 'Arrêter' : 'Enregistrer'}
      </button>
      {audioURL && (
        <audio controls src={audioURL} className="mt-4" />
      )}
    </div>
  );
}

// app/layout.tsx
export default function RootLayout({ children }) {
  return (
    <html lang="fr">
      <body className="p-4 font-sans">{children}</body>
    </html>
  );
}

// app/page.tsx (Accueil)
'use client';
import { login } from '../lib/auth';

export default function HomePage() {
  return (
    <div className="flex flex-col items-center gap-4">
      <h1 className="text-2xl font-bold">Mémoire Vivante</h1>
      <button onClick={login} className="bg-green-600 text-white px-4 py-2 rounded">Connexion Google</button>
    </div>
  );
}

// app/dashboard/page.tsx
'use client';
import { logout } from '../../lib/auth';

export default function Dashboard() {
  return (
    <div className="p-4">
      <h2 className="text-xl">Bienvenue</h2>
      <button onClick={logout} className="bg-red-500 text-white px-4 py-2 rounded">Déconnexion</button>
    </div>
  );
}

// app/record/page.tsx
import AudioRecorder from '../../components/AudioRecorder';

export default function RecordPage() {
  return (
    <div className="p-4">
      <h2 className="text-xl font-semibold mb-4">Enregistrement vocal</h2>
      <AudioRecorder />
    </div>
  );
}

// === 3. .env.local (non commité)
// NEXT_PUBLIC_FIREBASE_API_KEY=...
// NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN=...
// NEXT_PUBLIC_FIREBASE_PROJECT_ID=...
