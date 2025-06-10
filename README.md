# Me
Journey to learning and understanding myself
import { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import { Switch } from "@/components/ui/switch";
import { initializeApp } from "firebase/app";
import {
  getFirestore,
  doc,
  setDoc,
  getDoc,
  collection,
  addDoc,
  onSnapshot
} from "firebase/firestore";
import Image from "next/image";
import { motion } from "framer-motion";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_AUTH_DOMAIN",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_STORAGE_BUCKET",
  messagingSenderId: "YOUR_MESSAGING_SENDER_ID",
  appId: "YOUR_APP_ID"
};

const app = initializeApp(firebaseConfig);
const db = getFirestore(app);

const chapters = [
  {
    title: "Cracks in the Ordinary",
    quote: "Ezra’s father stopped watching the news... But something about it made him curious.",
    image: "/images/cracks.jpg",
    music: "/music/chapter1.mp3",
    prompts: [
      "Where in your life do you feel something is starting to shift?",
      "What moments recently have made you pause and wonder?"
    ]
  },
  {
    title: "Letting Go",
    quote: "James let go of old friends. Let go of a job. Let go of pretending... Ezra felt like he was losing the dad he knew.",
    image: "/images/letting-go.jpg",
    music: "/music/chapter2.mp3",
    prompts: [
      "What are you holding onto that may no longer serve you?",
      "What might be waiting if you created space?"
    ]
  },
  {
    title: "Questions with No Answers",
    quote: "\"Why are we here?\" James asked the air. Ezra didn’t answer. He just listened.",
    image: "/images/questions.jpg",
    music: "/music/chapter3.mp3",
    prompts: [
      "What big questions are living inside you right now?",
      "Can you sit with them without needing to solve them?"
    ]
  },
  {
    title: "Getting Out of Your Head",
    quote: "\"You think too much,\" James said gently. \"Come outside with me.\" They walked barefoot.",
    image: "/images/presence.jpg",
    music: "/music/chapter4.mp3",
    prompts: [
      "When was the last time you felt fully present?",
      "What helps you get out of your thoughts and into your senses?"
    ]
  },
  {
    title: "Signs in You",
    quote: "Ezra began noticing small things... One day he whispered, \"I think it’s starting in me too.\"",
    image: "/images/awakening.jpg",
    music: "/music/chapter5.mp3",
    prompts: [
      "What part of you feels like it’s beginning to awaken?",
      "What signs have you started noticing?"
    ]
  }
];

export default function SoulStirsJournal() {
  const [entries, setEntries] = useState({});
  const [userId, setUserId] = useState("");
  const [sharedEntries, setSharedEntries] = useState([]);
  const [sharePublicly, setSharePublicly] = useState(false);
  const [darkMode, setDarkMode] = useState(false);
  const [voiceNotes, setVoiceNotes] = useState({});

  useEffect(() => {
    const unsubscribe = onSnapshot(collection(db, "sharedEntries"), (snapshot) => {
      const data = snapshot.docs.map((doc) => doc.data());
      setSharedEntries(data);
    });
    return () => unsubscribe();
  }, []);

  const handleChange = (chapterIndex, promptIndex, value) => {
    setEntries((prev) => ({
      ...prev,
      [`${chapterIndex}-${promptIndex}`]: value,
    }));
  };

  const handleSave = async () => {
    const blob = new Blob([JSON.stringify(entries, null, 2)], { type: "application/json" });
    const url = URL.createObjectURL(blob);
    const link = document.createElement("a");
    link.href = url;
    link.download = "soul-stirs-journal.json";
    link.click();
  };

  const handleOnlineSave = async () => {
    if (!userId) return alert("Please enter a User ID to save your work.");
    await setDoc(doc(db, "journals", userId), { entries, voiceNotes });
    alert("🌿 Your journal has been saved to the cloud.");
  };

  const handleLoad = async () => {
    if (!userId) return alert("Please enter a User ID to load your work.");
    const docRef = doc(db, "journals", userId);
    const docSnap = await getDoc(docRef);
    if (docSnap.exists()) {
      const data = docSnap.data();
      setEntries(data.entries || {});
      setVoiceNotes(data.voiceNotes || {});
      alert("📖 Your journal has been retrieved. Welcome back.");
    } else {
      alert("No journal found for this ID.");
    }
  };

  const handleShare = async () => {
    if (sharePublicly) {
      await addDoc(collection(db, "sharedEntries"), {
        userId,
        timestamp: Date.now(),
        entries
      });
      alert("✨ Your reflection is now part of the collective wisdom.");
    } else {
      alert("🧘‍♂️ Your entry stays safe and private.");
    }
  };

  const handleVoiceUpload = (chapterIndex, file) => {
    const reader = new FileReader();
    reader.onload = () => {
      setVoiceNotes((prev) => ({
        ...prev,
        [chapterIndex]: reader.result
      }));
    };
    reader.readAsDataURL(file);
  };

  const completedChapters = Object.keys(entries).reduce((acc, key) => {
    const [chapterIdx] = key.split("-");
    acc.add(chapterIdx);
    return acc;
  }, new Set());

  return (
    <div className={darkMode ? "bg-gray-900 text-white min-h-screen" : "bg-white text-black min-h-screen"}>
      <motion.div
        className="max-w-3xl mx-auto p-6 space-y-8"
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        transition={{ duration: 1 }}
      >
        <h1 className="text-4xl font-bold text-center">When the Soul Stirs</h1>
        <p className="text-center text-muted-foreground">An Interactive Journal Inspired by Ezra's Story</p>

        <div className="flex items-center justify-between mt-4">
          <div className="flex gap-2 items-center">
            <Input
              placeholder="Enter your User ID"
              value={userId}
              onChange={(e) => setUserId(e.target.value)}
              className="w-48"
            />
            <Button onClick={handleLoad}>Load Journal</Button>
            <Button onClick={handleOnlineSave}>Save Online</Button>
          </div>
          <div className="flex items-center gap-2">
            <label>Dark Mode</label>
            <Switch checked={darkMode} onCheckedChange={setDarkMode} />
          </div>
        </div>

        <div className="w-full bg-gray-300 h-2 rounded-full overflow-hidden mt-4">
          <div
            className="bg-green-500 h-full transition-all duration-500"
            style={{ width: `${(completedChapters.size / chapters.length) * 100}%` }}
          ></div>
        </div>
        <p className="text-sm text-center mt-1">{completedChapters.size} of {chapters.length} chapters completed</p>

        {chapters.map((chapter, chapterIndex) => (
          <motion.div
            key={chapterIndex}
            initial={{ opacity: 0, y: 40 }}
            whileInView={{ opacity: 1, y: 0 }}
            transition={{ duration: 1 }}
          >
            <Card className="p-4 mt-6">
              <CardContent>
                <h2 className="text-2xl font-semibold mb-2">{chapter.title}</h2>
                <blockquote className="italic mb-4">“{chapter.quote}”</blockquote>
                {chapter.image && (
                  <Image
                    src={chapter.image}
                    alt={chapter.title}
                    width={800}
                    height={400}
                    className="rounded-xl mb-4 shadow-lg"
                  />
                )}
                <audio controls src={chapter.music} className="mb-4 w-full" />
                {chapter.prompts.map((prompt, promptIndex) => (
                  <div key={promptIndex} className="mb-4">
                    <label className="block mb-2 font-medium">{prompt}</label>
                    <Textarea
                      rows={4}
                      value={entries[`${chapterIndex}-${promptIndex}`] || ""}
                      onChange={(e) => handleChange(chapterIndex, promptIndex, e.target.value)}
                      placeholder="Write your reflection here..."
                    />
                  </div>
                ))}
                <div className="mt-2">
                  <label className="block mb-1 font-medium">Upload a voice note:</label>
                  <Input
                    type="file"
                    accept="audio/*"
                    onChange={(e) => handleVoiceUpload(chapterIndex, e.target.files[0])}
                  />
                </div>
              </CardContent>
            </Card>
          </motion.div>
        ))}

        <div className="text-center space-x-2 mt-6">
          <Button onClick={handleSave}>Download My Journal</Button>
          <Button onClick={handleShare}>Share Entry</Button>
          <label className="inline-flex items-center ml-4">
            <input
              type="checkbox"
              checked={sharePublicly}
              onChange={() => setSharePublicly(!sharePublicly)}
              className="mr-2"
            />
            Share with community
          </label>
        </div>

        <div className="mt-10">
          <h2 className="text-xl font-bold mb-2">🌍 Community Reflections</h2>
          {sharedEntries.map((entry, idx) => (
            <div key={idx} className="bg-gray-100 rounded-xl p-4 mb-4 text-sm">
              <pre className="whitespace-pre-wrap">{JSON.stringify(entry.entries, null, 2)}</pre>
            </div>
          ))}
        </div>
      </motion.div>
    </div>
  );
} 
