# promptmaster-pro
git init git add . git commit -m "Initial commit" git remote add origin https://github.com/VOTRE_USERNAME/promptmaster-pro.git git push -u origin main
import React, { useState, useEffect } from 'react';
import { Sparkles, Copy, Check, Wand2, Zap, Crown, Download, History, Trash2, Star, Share2, Languages, Shuffle, BookOpen } from 'lucide-react';

export default function PromptMasterPro() {
  const [input, setInput] = useState('');
  const [prompts, setPrompts] = useState(null);
  const [loading, setLoading] = useState(false);
  const [copied, setCopied] = useState(null);
  const [history, setHistory] = useState([]);
  const [showHistory, setShowHistory] = useState(false);
  const [favorites, setFavorites] = useState([]);
  const [showFavorites, setShowFavorites] = useState(false);
  const [language, setLanguage] = useState('fr');
  const [showTemplates, setShowTemplates] = useState(false);
  const [aiMode, setAiMode] = useState('chatgpt');

  const templates = [
    { name: 'Email Marketing', desc: 'Email de vente produit/service', prompt: 'email de vente pour [produit] avec accroche persuasive' },
    { name: 'Contenu LinkedIn', desc: 'Post viral LinkedIn', prompt: 'post LinkedIn viral sur [sujet] pour entrepreneurs' },
    { name: 'Article de Blog', desc: 'Article SEO optimisé', prompt: 'article de blog 1500 mots sur [sujet] optimisé SEO' },
    { name: 'Copywriting Pub', desc: 'Annonce publicitaire', prompt: 'annonce Facebook Ads pour [produit] avec hook émotionnel' },
    { name: 'Script Vidéo', desc: 'Script YouTube/TikTok', prompt: 'script vidéo 60 secondes pour [sujet] format storytelling' },
    { name: 'Pitch Startup', desc: 'Pitch investisseurs', prompt: 'pitch deck startup [domaine] pour lever fonds' },
    { name: 'Fiche Produit', desc: 'Description e-commerce', prompt: 'fiche produit e-commerce [produit] avec bénéfices clients' },
    { name: 'ChatBot Support', desc: 'Réponses automatiques', prompt: 'réponses chatbot support client [entreprise] ton amical' }
  ];

  useEffect(() => {
    loadHistory();
    loadFavorites();
  }, []);

  const loadHistory = async () => {
    try {
      const keys = await window.storage.list('prompt:', false);
      if (keys && keys.keys) {
        const historyItems = [];
        for (const key of keys.keys.slice(0, 15)) {
          try {
            const result = await window.storage.get(key, false);
            if (result) {
              historyItems.push(JSON.parse(result.value));
            }
          } catch (e) {
            console.log('Key not found:', key);
          }
        }
        setHistory(historyItems.sort((a, b) => b.timestamp - a.timestamp));
      }
    } catch (error) {
      console.log('No history yet');
    }
  };

  const loadFavorites = async () => {
    try {
      const keys = await window.storage.list('favorite:', false);
      if (keys && keys.keys) {
        const favItems = [];
        for (const key of keys.keys) {
          try {
            const result = await window.storage.get(key, false);
            if (result) {
              favItems.push(JSON.parse(result.value));
            }
          } catch (e) {
            console.log('Key not found:', key);
          }
        }
        setFavorites(favItems.sort((a, b) => b.timestamp - a.timestamp));
      }
    } catch (error) {
      console.log('No favorites yet');
    }
  };

  const saveToHistory = async (userInput, generatedPrompts) => {
    const historyItem = {
      id: Date.now().toString(),
      input: userInput,
      prompts: generatedPrompts,
      timestamp: Date.now(),
      language,
      aiMode
    };
    
    try {
      await window.storage.set(`prompt:${historyItem.id}`, JSON.stringify(historyItem), false);
      await loadHistory();
    } catch (error) {
      console.error('Error saving history:', error);
    }
  };

  const toggleFavorite = async (item, level) => {
    const favId = `${item.id}_${level}`;
    const existingFav = favorites.find(f => f.id === favId);
    
    if (existingFav) {
      try {
        await window.storage.delete(`favorite:${favId}`, false);
        await loadFavorites();
      } catch (error) {
        console.error('Error removing favorite:', error);
      }
    } else {
      const favorite = {
        id: favId,
        originalId: item.id,
        input: item.input,
        prompt: item.prompts[level],
        level,
        timestamp: Date.now()
      };
      try {
        await window.storage.set(`favorite:${favId}`, JSON.stringify(favorite), false);
        await loadFavorites();
      } catch (error) {
        console.error('Error saving favorite:', error);
      }
    }
  };

  const isFavorite = (itemId, level) => {
    return favorites.some(f => f.id === `${itemId}_${level}`);
  };

  const deleteHistoryItem = async (id) => {
    try {
      await window.storage.delete(`prompt:${id}`, false);
      await loadHistory();
    } catch (error) {
      console.error('Error deleting history:', error);
    }
  };

  const deleteFavorite = async (id) => {
    try {
      await window.storage.delete(`favorite:${id}`, false);
      await loadFavorites();
    } catch (error) {
      console.error('Error deleting favorite:', error);
    }
  };

  const getRandomTemplate = () => {
    const randomTemplate = templates[Math.floor(Math.random() * templates.length)];
    setInput(randomTemplate.prompt);
    setShowTemplates(false);
  };

  const generatePrompts = async () => {
    if (!input.trim()) return;

    setLoading(true);
    setPrompts(null);

    const languageInstruction = language === 'en' ? 'Respond in English.' : 
                               language === 'es' ? 'Responde en español.' : 
                               language === 'de' ? 'Antworte auf Deutsch.' : 
                               'Réponds en français.';

    const aiModeContext = aiMode === 'chatgpt' ? 'optimisés pour ChatGPT (GPT-4)' :
                         aiMode === 'claude' ? 'optimisés pour Claude (Anthropic)' :
                         aiMode === 'gemini' ? 'optimisés pour Gemini (Google)' :
                         'universels compatibles toutes IA';

    const systemPrompt = `Tu es PROMPTMASTER, expert en génération de prompts IA optimisés.

MISSION : Transformer l'idée de l'utilisateur en 3 prompts progressifs (Simple, Avancé, Expert) ${aiModeContext}.

${languageInstruction}

STRUCTURE OBLIGATOIRE pour chaque niveau :
- SIMPLE : Direct, 1-2 phrases, utilisable immédiatement
- AVANCÉ : Ajoute contexte, rôle, format, contraintes (50-100 mots)
- EXPERT : Maximum détail avec exemples, format JSON si pertinent, méthodologie (100-200 mots)

RÈGLES :
1. Analyse l'intention réelle derrière la demande
2. Ajoute toujours : rôle expert, contexte, format sortie
3. Optimise pour clarté et reproductibilité
4. ${languageInstruction}
5. Réponds UNIQUEMENT avec les 3 prompts, pas d'explications additionnelles

FORMAT STRICT :
🎯 SIMPLE
[prompt simple]

🎯 AVANCÉ
[prompt avancé]

🎯 EXPERT
[prompt expert]`;

    try {
      const response = await fetch('https://api.anthropic.com/v1/messages', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          model: 'claude-sonnet-4-20250514',
          max_tokens: 2000,
          messages: [
            {
              role: 'user',
              content: `Génère 3 prompts optimisés pour : "${input}"`
            }
          ],
          system: systemPrompt
        })
      });

      const data = await response.json();
      const text = data.content.map(item => item.type === 'text' ? item.text : '').join('\n');
      
      const simple = text.match(/🎯 SIMPLE\n([\s\S]*?)(?=\n🎯 AVANCÉ|$)/)?.[1]?.trim() || '';
      const avance = text.match(/🎯 AVANCÉ\n([\s\S]*?)(?=\n🎯 EXPERT|$)/)?.[1]?.trim() || '';
      const expert = text.match(/🎯 EXPERT\n([\s\S]*?)$/)?.[1]?.trim() || '';

      const generatedPrompts = { simple, avance, expert };
      setPrompts(generatedPrompts);
      await saveToHistory(input, generatedPrompts);
      
    } catch (error) {
      console.error('Error:', error);
      setPrompts({
        simple: 'Erreur de génération. Veuillez réessayer.',
        avance: '',
        expert: ''
      });
    }

    setLoading(false);
  };

  const copyToClipboard = (text, level) => {
    navigator.clipboard.writeText(text);
    setCopied(level);
    setTimeout(() => setCopied(null), 2000);
  };

  const sharePrompt = async (text) => {
    if (navigator.share) {
      try {
        await navigator.share({
          title: 'Prompt IA optimisé',
          text: text
        });
      } catch (error) {
        copyToClipboard(text, 'share');
      }
    } else {
      copyToClipboard(text, 'share');
    }
  };

  const loadFromHistory = (item) => {
    setInput(item.input);
    setPrompts(item.prompts);
    setShowHistory(false);
    if (item.language) setLanguage(item.language);
    if (item.aiMode) setAiMode(item.aiMode);
  };

  const exportPrompts = () => {
    if (!prompts) return;
    const text = `🎯 PROMPT SIMPLE\n${prompts.simple}\n\n🎯 PROMPT AVANCÉ\n${prompts.avance}\n\n🎯 PROMPT EXPERT\n${prompts.expert}`;
    const blob = new Blob([text], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'prompts-generated.txt';
    a.click();
  };

  return (
    <div className="min-h-screen bg-gradient-to-br from-purple-900 via-indigo-900 to-blue-900">
      <div className="bg-black/20 backdrop-blur-sm border-b border-white/10">
        <div className="max-w-6xl mx-auto px-4 py-4 flex items-center justify-between flex-wrap gap-3">
          <div className="flex items-center gap-3">
            <div className="bg-gradient-to-br from-yellow-400 to-orange-500 p-2 rounded-lg">
              <Sparkles className="w-6 h-6 text-white" />
            </div>
            <div>
              <h1 className="text-2xl font-bold text-white">PromptMaster Pro</h1>
              <p className="text-xs text-purple-200">Générateur de prompts IA optimisés</p>
            </div>
          </div>
          <div className="flex items-center gap-2 flex-wrap">
            <button
              onClick={() => setShowTemplates(!showTemplates)}
              className="flex items-center gap-2 px-3 py-2 bg-white/10 hover:bg-white/20 rounded-lg text-white transition-colors text-sm"
            >
              <BookOpen className="w-4 h-4" />
              Templates
            </button>
            <button
              onClick={() => setShowFavorites(!showFavorites)}
              className="flex items-center gap-2 px-3 py-2 bg-white/10 hover:bg-white/20 rounded-lg text-white transition-colors text-sm"
            >
              <Star className="w-4 h-4" />
              Favoris ({favorites.length})
            </button>
            <button
              onClick={() => setShowHistory(!showHistory)}
              className="flex items-center gap-2 px-3 py-2 bg-white/10 hover:bg-white/20 rounded-lg text-white transition-colors text-sm"
            >
              <History className="w-4 h-4" />
              Historique ({history.length})
            </button>
          </div>
        </div>
      </div>

      <div className="max-w-6xl mx-auto px-4 py-8">
        {showTemplates && (
          <div className="mb-6 bg-white/10 backdrop-blur-md rounded-2xl p-6 border border-white/20">
            <div className="flex items-center justify-between mb-4">
              <h3 className="text-white font-semibold flex items-center gap-2">
                <BookOpen className="w-5 h-5" />
                Templates prêts à l'emploi
              </h3>
              <button
                onClick={getRandomTemplate}
                className="flex items-center gap-2 px-3 py-2 bg-purple-500 hover:bg-purple-600 rounded-lg text-white text-sm transition-colors"
              >
                <Shuffle className="w-4 h-4" />
                Aléatoire
              </button>
            </div>
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-3">
              {templates.map((template, idx) => (
                <button
                  key={idx}
                  onClick={() => {
                    setInput(template.prompt);
                    setShowTemplates(false);
                  }}
                  className="bg-white/5 hover:bg-white/10 rounded-lg p-4 text-left transition-colors group"
                >
                  <p className="text-white font-semibold text-sm mb-1">{template.name}</p>
                  <p className="text-purple-200 text-xs">{template.desc}</p>
                </button>
              ))}
            </div>
          </div>
        )}

        {showFavorites && (
          <div className="mb-6 bg-white/10 backdrop-blur-md rounded-2xl p-4 border border-white/20">
            <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
              <Star className="w-5 h-5 text-yellow-400" />
              Prompts favoris
            </h3>
            <div className="space-y-2 max-h-96 overflow-y-auto">
              {favorites.map(fav => (
                <div
                  key={fav.id}
                  className="bg-white/5 hover:bg-white/10 rounded-lg p-3 transition-colors group"
                >
                  <div className="flex items-start justify-between gap-2 mb-2">
                    <span className={`text-xs px-2 py-1 rounded-full ${
                      fav.level === 'simple' ? 'bg-green-500/20 text-green-300' :
                      fav.level === 'avance' ? 'bg-blue-500/20 text-blue-300' :
                      'bg-purple-500/20 text-purple-300'
                    }`}>
                      {fav.level.toUpperCase()}
                    </span>
                    <button
                      onClick={() => deleteFavorite(fav.id)}
                      className="opacity-0 group-hover:opacity-100 p-1 hover:bg-red-500/20 rounded transition-all"
                    >
                      <Trash2 className="w-4 h-4 text-red-400" />
                    </button>
                  </div>
                  <p className="text-white text-sm line-clamp-2 mb-2">{fav.prompt}</p>
                  <button
                    onClick={() => copyToClipboard(fav.prompt, fav.id)}
                    className="text-purple-300 text-xs hover:text-purple-200 flex items-center gap-1"
                  >
                    {copied === fav.id ? <Check className="w-3 h-3" /> : <Copy className="w-3 h-3" />}
                    Copier
                  </button>
                </div>
              ))}
              {favorites.length === 0 && (
                <p className="text-purple-300 text-sm text-center py-4">Aucun favori pour le moment</p>
              )}
            </div>
          </div>
        )}

        {showHistory && (
          <div className="mb-6 bg-white/10 backdrop-blur-md rounded-2xl p-4 border border-white/20">
            <h3 className="text-white font-semibold mb-4 flex items-center gap-2">
              <History className="w-5 h-5" />
              Historique récent
            </h3>
            <div className="space-y-2 max-h-96 overflow-y-auto">
              {history.map(item => (
                <div
                  key={item.id}
                  className="bg-white/5 hover:bg-white/10 rounded-lg p-3 cursor-pointer transition-colors group"
                >
                  <div className="flex items-start justify-between gap-2">
                    <div className="flex-1" onClick={() => loadFromHistory(item)}>
                      <p className="text-white text-sm font-medium line-clamp-1">{item.input}</p>
                      <div className="flex items-center gap-2 mt-1">
                        <p className="text-purple-200 text-xs">
                          {new Date(item.timestamp).toLocaleDateString('fr-FR')}
                        </p>
                        {item.aiMode && (
                          <span className="text-xs px-2 py-0.5 bg-white/10 rounded text-purple-200">
                            {item.aiMode}
                          </span>
                        )}
                      </div>
                    </div>
                    <button
                      onClick={(e) => {
                        e.stopPropagation();
                        deleteHistoryItem(item.id);
                      }}
                      className="opacity-0 group-hover:opacity-100 p-1 hover:bg-red-500/20 rounded transition-all"
                    >
                      <Trash2 className="w-4 h-4 text-red-400" />
                    </button>
                  </div>
                </div>
              ))}
              {history.length === 0 && (
                <p className="text-purple-300 text-sm text-center py-4">Aucun historique</p>
              )}
            </div>
          </div>
        )}

        <div className="bg-white/10 backdrop-blur-md rounded-2xl p-6 mb-6 border border-white/20 shadow-2xl">
          <label className="block text-white font-semibold mb-3 flex items-center gap-2">
            <Wand2 className="w-5 h-5" />
            Décrivez votre besoin
          </label>
          
          <div className="flex items-center gap-3 mb-4 flex-wrap">
            <div className="flex items-center gap-2">
              <Languages className="w-4 h-4 text-purple-300" />
              <select
                value={language}
                onChange={(e) => setLanguage(e.target.value)}
                className="px-3 py-1.5 bg-white/10 border border-white/20 rounded-lg text-white text-sm focus:outline-none focus:ring-2 focus:ring-purple-500"
              >
                <option value="fr">🇫🇷 Français</option>
                <option value="en">🇬🇧 English</option>
                <option value="es">🇪🇸 Español</option>
                <option value="de">🇩🇪 Deutsch</option>
              </select>
            </div>
            
            <div className="flex items-center gap-2">
              <Sparkles className="w-4 h-4 text-purple-300" />
              <select
                value={aiMode}
                onChange={(e) => setAiMode(e.target.value)}
                className="px-3 py-1.5 bg-white/10 border border-white/20 rounded-lg text-white text-sm focus:outline-none focus:ring-2 focus:ring-purple-500"
              >
                <option value="chatgpt">ChatGPT</option>
                <option value="claude">Claude</option>
                <option value="gemini">Gemini</option>
                <option value="universal">Universel</option>
              </select>
            </div>
          </div>

          <textarea
            value={input}
            onChange={(e) => setInput(e.target.value)}
            onKeyDown={(e) => {
              if (e.key === 'Enter' && e.ctrlKey) {
                generatePrompts();
              }
            }}
            placeholder='Ex: "générer des idées de posts LinkedIn pour coach business"'
            className="w-full px-4 py-3 bg-white/10 border border-white/20 rounded-xl text-white placeholder-purple-300 focus:outline-none focus:ring-2 focus:ring-purple-500 resize-none"
            rows="3"
          />
          <div className="flex items-center justify-between mt-4">
            <p className="text-purple-200 text-sm">
              Ctrl + Entrée pour générer
            </p>
            <button
              onClick={generatePrompts}
              disabled={loading || !input.trim()}
              className="flex items-center gap-2 px-6 py-3 bg-gradient-to-r from-purple-500 to-pink-500 hover:from-purple-600 hover:to-pink-600 disabled:opacity-50 disabled:cursor-not-allowed rounded-xl text-white font-semibold transition-all transform hover:scale-105 shadow-lg"
            >
              {loading ? (
                <>
                  <div className="w-5 h-5 border-2 border-white/30 border-t-white rounded-full animate-spin" />
                  Génération...
                </>
              ) : (
                <>
                  <Sparkles className="w-5 h-5" />
                  Générer
                </>
              )}
            </button>
          </div>
        </div>

        {prompts && (
          <div className="space-y-4">
            <div className="flex items-center justify-between mb-2">
              <h2 className="text-white text-xl font-bold flex items-center gap-2">
                <Zap className="w-6 h-6 text-yellow-400" />
                Vos prompts optimisés
              </h2>
              <button
                onClick={exportPrompts}
                className="flex items-center gap-2 px-4 py-2 bg-white/10 hover:bg-white/20 rounded-lg text-white text-sm transition-colors"
              >
                <Download className="w-4 h-4" />
                Exporter
              </button>
            </div>

            <div className="bg-gradient-to-br from-green-500/20 to-emerald-500/20 backdrop-blur-md rounded-2xl p-5 border border-green-400/30 shadow-xl">
              <div className="flex items-start justify-between mb-3">
                <div className="flex items-center gap-2">
                  <div className="bg-green-500 text-white px-3 py-1 rounded-full text-sm font-semibold">
                    🎯 SIMPLE
                  </div>
                  <span className="text-green-200 text-sm">Débutant</span>
                </div>
                <div className="flex items-center gap-2">
                  <button
                    onClick={() => toggleFavorite({id: 'current', input, prompts}, 'simple')}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    <Star className={`w-5 h-5 ${isFavorite('current', 'simple') ? 'fill-yellow-400 text-yellow-400' : 'text-white'}`} />
                  </button>
                  <button
                    onClick={() => sharePrompt(prompts.simple)}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    <Share2 className="w-5 h-5 text-white" />
                  </button>
                  <button
                    onClick={() => copyToClipboard(prompts.simple, 'simple')}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    {copied === 'simple' ? (
                      <Check className="w-5 h-5 text-green-400" />
                    ) : (
                      <Copy className="w-5 h-5 text-white" />
                    )}
                  </button>
                </div>
              </div>
              <p className="text-white leading-relaxed whitespace-pre-wrap">{prompts.simple}</p>
            </div>

            <div className="bg-gradient-to-br from-blue-500/20 to-cyan-500/20 backdrop-blur-md rounded-2xl p-5 border border-blue-400/30 shadow-xl">
              <div className="flex items-start justify-between mb-3">
                <div className="flex items-center gap-2">
                  <div className="bg-blue-500 text-white px-3 py-1 rounded-full text-sm font-semibold">
                    🎯 AVANCÉ
                  </div>
                  <span className="text-blue-200 text-sm">Pro</span>
                </div>
                <div className="flex items-center gap-2">
                  <button
                    onClick={() => toggleFavorite({id: 'current', input, prompts}, 'avance')}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    <Star className={`w-5 h-5 ${isFavorite('current', 'avance') ? 'fill-yellow-400 text-yellow-400' : 'text-white'}`} />
                  </button>
                  <button
                    onClick={() => sharePrompt(prompts.avance)}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    <Share2 className="w-5 h-5 text-white" />
                  </button>
                  <button
                    onClick={() => copyToClipboard(prompts.avance, 'avance')}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    {copied === 'avance' ? (
                      <Check className="w-5 h-5 text-blue-400" />
                    ) : (
                      <Copy className="w-5 h-5 text-white" />
                    )}
                  </button>
                </div>
              </div>
              <p className="text-white leading-relaxed whitespace-pre-wrap">{prompts.avance}</p>
            </div>

            <div className="bg-gradient-to-br from-purple-500/20 to-pink-500/20 backdrop-blur-md rounded-2xl p-5 border border-purple-400/30 shadow-xl">
              <div className="flex items-start justify-between mb-3">
                <div className="flex items-center gap-2">
                  <div className="bg-gradient-to-r from-purple-500 to-pink-500 text-white px-3 py-1 rounded-full text-sm font-semibold flex items-center gap-1">
                    <Crown className="w-4 h-4" />
                    🎯 EXPERT
                  </div>
                  <span className="text-purple-200 text-sm">Maximum</span>
                </div>
                <div className="flex items-center gap-2">
                  <button
                    onClick={() => toggleFavorite({id: 'current', input, prompts}, 'expert')}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    <Star className={`w-5 h-5 ${isFavorite('current', 'expert') ? 'fill-yellow-400 text-yellow-400' : 'text-white'}`} />
                  </button>
                  <button
                    onClick={() => sharePrompt(prompts.expert)}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    <Share2 className="w-5 h-5 text-white" />
                  </button>
                  <button
                    onClick={() => copyToClipboard(prompts.expert, 'expert')}
                    className="p-2 hover:bg-white/10 rounded-lg transition-colors"
                  >
                    {copied === 'expert' ? (
                      <Check className="w-5 h-5 text-purple-400" />
                    ) : (
                      <Copy className="w-5 h-5 text-white" />
                    )}
                  </button>
                </div>
              </div>
              <p className="text-white leading-relaxed whitespace-pre-wrap">{prompts.expert}</p>
            </div>
          </div>
        )}

        {!prompts && !loading && (
          <div className="text-center py-16">
            <div className="bg-white/5 backdrop-blur-sm rounded-2xl p-12 border border-white/10">
              <Sparkles className="w-16 h-16 text-purple-400 mx-auto mb-4" />
              <h3 className="text-white text-xl font-semibold mb-2">
                Prêt à créer des prompts parfaits ?
              </h3>
              <p className="text-purple-200 mb-6">
                Décrivez votre besoin et obtenez 3 prompts optimisés en quelques secondes
              </p>
              <div className="grid grid-cols-1 md:grid-cols-3 gap-4 max-w-3xl mx-auto text-left">
                <div className="bg-white/5 p-4 rounded-lg">
                  <p className="text-green-400 font-semibold mb-1">Simple</p>
                  <p className="text-purple-200 text-sm">Pour démarrer rapidement</p>
                </div>
                <div className="bg-white/5 p-4 rounded-lg">
                  <p className="text-blue-400 font-semibold mb-1">Avancé</p>
                  <p className="text-purple-200 text-sm">Avec contexte détaillé</p>
                </div>
                <div className="bg-white/5 p-4 rounded-lg">
                  <p className="text-purple-400 font-semibold mb-1">Expert</p>
                  <p className="text-purple-200 text-sm">Maximum de précision</p>
                </div>
              </div>
            </div>
          </div>
        )}
      </div>

      <div className="text-center py-6 text-purple-300 text-sm">
        <p>Propulsé par Claude AI • {favorites.length} favoris • {history.length} générations • Multi-langue</p>
      </div>
    </div>
  );
}
