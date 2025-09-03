import React, { useEffect, useMemo, useRef, useState } from "react";
import { motion } from "framer-motion";
import {
  Area,
  AreaChart,
  CartesianGrid,
  Legend,
  ResponsiveContainer,
  Tooltip,
  XAxis,
  YAxis,
} from "recharts";
import { Button } from "@/components/ui/button";
import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Tabs, TabsContent, TabsList, TabsTrigger } from "@/components/ui/tabs";
import { Badge } from "@/components/ui/badge";
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from "@/components/ui/select";
import { Separator } from "@/components/ui/separator";
import { ScrollArea } from "@/components/ui/scroll-area";
import {
  ArrowUpRight,
  ArrowDownRight,
  Bot,
  LineChart,
  Search,
  RefreshCcw,
  Star,
  Sparkles,
  TrendingUp,
  SunMoon,
  ExternalLink,
} from "lucide-react";

// -----------------------------------------------------------------------------
// Utilities and mock data
// -----------------------------------------------------------------------------

const fmt = new Intl.NumberFormat("de-DE", { maximumFractionDigits: 2 });

function generateSeries(label = "STOCK") {
  const data = [];
  let price = 100 + Math.random() * 50;
  const today = new Date();
  for (let i = 120; i >= 0; i--) {
    const d = new Date(today);
    d.setDate(d.getDate() - i);
    // random walk
    const drift = Math.sin(i / 9) * 0.2 + (Math.random() - 0.5) * 0.8;
    price = Math.max(10, price + drift);
    data.push({
      date: d.toISOString().slice(0, 10),
      price: +price.toFixed(2),
      label,
    });
  }
  return data;
}

function movingAverage(arr, window = 14) {
  const out = [];
  for (let i = 0; i < arr.length; i++) {
    const start = Math.max(0, i - window + 1);
    const slice = arr.slice(start, i + 1);
    const avg = slice.reduce((a, b) => a + b.price, 0) / slice.length;
    out.push({ ...arr[i], ma: +avg.toFixed(2) });
  }
  return out;
}

function naiveForecast(lastPrice, months = 6) {
  // Extremely naive forecast for demo only. Replace with your own model.
  // Uses small random drift around last known price.
  const points = [];
  const start = new Date();
  for (let m = 1; m <= months; m++) {
    const d = new Date(start);
    d.setMonth(d.getMonth() + m);
    const drift = (Math.random() - 0.4) * 6; // bias slightly upward
    const p = Math.max(5, lastPrice + drift);
    points.push({ date: d.toISOString().slice(0, 10), forecast: +p.toFixed(2) });
  }
  return points;
}

// -----------------------------------------------------------------------------
// AI Advisor stub
// -----------------------------------------------------------------------------
async function advisorReply(userText, context) {
  // Replace this stub with your own model call
  // Example OpenAI compatible call:
  // const res = await fetch("/api/ai", { method: "POST", body: JSON.stringify({
  //   messages: [
  //     { role: "system", content: "You are a cautious financial research assistant. Never give individualized advice. Include disclaimers." },
  //     { role: "user", content: userText },
  //     { role: "assistant", content: JSON.stringify(context) },
  //   ]
  // }) )
  // const data = await res.json();
  // return data.text;

  const tipPool = [
    "Dies ist kein Anlagehinweis. Prüfe Kosten, Steuern und Risikotragfähigkeit.",
    "Diversifikation mindert idiosynkratisches Risiko. Einzeltitel sind volatil.",
    "Historische Renditen sind keine Garantie. Szenarien statt Punktprognosen nutzen.",
    "Achte auf Gebühren von Brokern und Fonds sowie auf Währungsrisiken.",
  ];
  const pick = tipPool[Math.floor(Math.random() * tipPool.length)];
  return `Kurzantwort: ${userText.slice(0, 140)}?\n\nHinweis: ${pick}`;
}

// -----------------------------------------------------------------------------
// Main App
// -----------------------------------------------------------------------------
export default function App() {
  const [dark, setDark] = useState(true);
  const [query, setQuery] = useState("");
  const [ticker, setTicker] = useState("ACME");
  const baseSeries = useMemo(() => generateSeries(ticker), [ticker]);
  const series = useMemo(() => movingAverage(baseSeries), [baseSeries]);
  const last = series[series.length - 1]?.price ?? 100;
  const fc = useMemo(() => naiveForecast(last, 6), [last]);
  const [watchlist, setWatchlist] = useState(["AAPL", "MSFT", "NVDA", "SPY", "TSLA"]);
  const [loading, setLoading] = useState(false);
  const [messages, setMessages] = useState([
    { role: "system", content: "Willkommen. Ich helfe beim Recherchieren rund um Aktien. Keine Anlageberatung." },
  ]);

  // Simulated refresh
  useEffect(() => {
    const id = setInterval(() => {
      // trigger re-render by toggling ticker temp to itself
      setTicker((t) => t);
    }, 15000);
    return () => clearInterval(id);
  }, []);

  useEffect(() => {
    if (dark) document.documentElement.classList.add("dark");
    else document.documentElement.classList.remove("dark");
  }, [dark]);

  async function handleSearch() {
    if (!query) return;
    setLoading(true);
    // In production: fetch real quotes and news here
    // Example: Alpha Vantage, Finnhub, Yahoo Finance
    setTimeout(() => {
      setTicker(query.toUpperCase());
      if (!watchlist.includes(query.toUpperCase()))
        setWatchlist((w) => [query.toUpperCase(), ...w].slice(0, 10));
      setQuery("");
      setLoading(false);
    }, 600);
  }

  async function handleAskAi(text) {
    const content = text || query;
    if (!content) return;
    setMessages((m) => [...m, { role: "user", content }]);
    const reply = await advisorReply(content, { ticker, last });
    setMessages((m) => [...m, { role: "user", content }, { role: "assistant", content: reply }]);
    setQuery("");
  }

  return (
    <div className="min-h-screen bg-gradient-to-b from-slate-950 via-slate-900 to-slate-950 text-slate-100">
      <div className="max-w-7xl mx-auto p-4 md:p-8">
        {/* Header */}
        <div className="flex items-center justify-between mb-6">
          <div className="flex items-center gap-3">
            <motion.div initial={{ scale: 0.9, opacity: 0 }} animate={{ scale: 1, opacity: 1 }}>
              <div className="p-3 rounded-2xl bg-slate-800 shadow-inner shadow-slate-900">
                <Sparkles className="h-6 w-6" />
              </div>
            </motion.div>
            <div>
              <h1 className="text-2xl md:text-3xl font-bold tracking-tight">Aktien KI Dashboard</h1>
              <p className="text-slate-400 text-sm">Recherche Visualisierung und KI Assistent. Bildungszwecke keine Anlageberatung</p>
            </div>
          </div>
          <div className="flex items-center gap-2">
            <Button variant="secondary" onClick={() => setDark((d) => !d)} className="rounded-2xl">
              <SunMoon className="h-4 w-4 mr-2" /> Theme
            </Button>
            <Button variant="outline" className="rounded-2xl" onClick={() => window.location.reload()}>
              <RefreshCcw className="h-4 w-4 mr-2" /> Refresh
            </Button>
          </div>
        </div>

        {/* Search and quick stats */}
        <div className="grid grid-cols-1 md:grid-cols-3 gap-4 mb-6">
          <Card className="rounded-2xl bg-slate-900 border-slate-800">
            <CardHeader className="pb-2">
              <CardTitle className="text-base flex items-center gap-2"><Search className="h-4 w-4" /> Ticker suchen</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="flex gap-2">
                <Input placeholder="zB AAPL NVDA SAP" value={query} onChange={(e) => setQuery(e.target.value)} onKeyDown={(e) => e.key === 'Enter' && handleSearch()} className="rounded-2xl bg-slate-950 border-slate-800" />
                <Button onClick={handleSearch} disabled={loading} className="rounded-2xl">Suchen</Button>
              </div>
              <div className="mt-3 text-sm text-slate-400">Letzter Ticker: <Badge variant="secondary" className="rounded-xl">{ticker}</Badge></div>
            </CardContent>
          </Card>

          <StatCard title="Kurs aktuell" value={`€ ${fmt.format(last)}`} trend={+((last - series.at(-2)?.price) || 0).toFixed(2)} />
          <StatCard title="6M Forecast Demo" value={`${fmt.format(fc.at(-1)?.forecast ?? 0)} pts`} trend={+(((fc.at(-1)?.forecast ?? 0) - last).toFixed(2))} />
        </div>

        {/* Main Content */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
          {/* Chart */}
          <Card className="rounded-2xl bg-slate-900 border-slate-800 lg:col-span-2">
            <CardHeader className="pb-2">
              <div className="flex items-center justify-between">
                <CardTitle className="text-base flex items-center gap-2"><LineChart className="h-4 w-4" /> {ticker} Kurs & MA</CardTitle>
                <div className="flex items-center gap-2 text-xs text-slate-400">
                  <Badge variant="outline" className="rounded-xl">Demo Daten</Badge>
                  <a className="inline-flex items-center gap-1 hover:underline" href="#" onClick={(e) => e.preventDefault()}>
                    <ExternalLink className="h-3 w-3" /> API verbinden
                  </a>
                </div>
              </div>
            </CardHeader>
            <CardContent>
              <div className="h-80">
                <ResponsiveContainer width="100%" height="100%">
                  <AreaChart data={series} margin={{ top: 10, right: 20, left: 0, bottom: 0 }}>
                    <defs>
                      <linearGradient id="g1" x1="0" y1="0" x2="0" y2="1">
                        <stop offset="5%" stopColor="#60a5fa" stopOpacity={0.35} />
                        <stop offset="95%" stopColor="#60a5fa" stopOpacity={0} />
                      </linearGradient>
                      <linearGradient id="g2" x1="0" y1="0" x2="0" y2="1">
                        <stop offset="5%" stopColor="#34d399" stopOpacity={0.25} />
                        <stop offset="95%" stopColor="#34d399" stopOpacity={0} />
                      </linearGradient>
                    </defs>
                    <CartesianGrid strokeDasharray="3 3" stroke="#1f2937" />
                    <XAxis dataKey="date" tick={{ fill: "#94a3b8", fontSize: 12 }} minTickGap={24} />
                    <YAxis tick={{ fill: "#94a3b8", fontSize: 12 }} domain={["auto", "auto"]} />
                    <Tooltip contentStyle={{ background: "#0b1220", border: "1px solid #1f2937" }} labelStyle={{ color: "#cbd5e1" }} />
                    <Legend />
                    <Area type="monotone" dataKey="price" name="Kurs" stroke="#60a5fa" fillOpacity={1} fill="url(#g1)" />
                    <Area type="monotone" dataKey="ma" name="MA14" stroke="#34d399" fillOpacity={1} fill="url(#g2)" />
                  </AreaChart>
                </ResponsiveContainer>
              </div>
              <Separator className="my-4 bg-slate-800" />
              <div>
                <h3 className="text-sm font-semibold mb-2">Naive 6 Monate Demo Forecast</h3>
                <div className="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-6 gap-2 text-xs">
                  {fc.map((p) => (
                    <div key={p.date} className="p-2 rounded-xl bg-slate-950 border border-slate-800">
                      <div className="text-slate-400">{p.date}</div>
                      <div className="font-semibold">{fmt.format(p.forecast)}</div>
                    </div>
                  ))}
                </div>
              </div>
            </CardContent>
          </Card>

          {/* AI Advisor */}
          <Card className="rounded-2xl bg-slate-900 border-slate-800">
            <CardHeader className="pb-2">
              <CardTitle className="text-base flex items-center gap-2"><Bot className="h-4 w-4" /> KI Assistent</CardTitle>
            </CardHeader>
            <CardContent>
              <div className="h-80 overflow-hidden rounded-xl border border-slate-800 bg-slate-950">
                <ScrollArea className="h-64 p-3">
                  <div className="space-y-3">
                    {messages.map((m, i) => (
                      <div key={i} className={"text-sm " + (m.role === "assistant" ? "text-slate-200" : m.role === "system" ? "text-slate-400" : "text-sky-300")}>{m.content}</div>
                    ))}
                  </div>
                </ScrollArea>
                <div className="p-2 flex gap-2 border-t border-slate-800">
                  <Input placeholder="Frage zur Aktie oder zum Markt" value={query} onChange={(e) => setQuery(e.target.value)} onKeyDown={(e) => e.key === 'Enter' && handleAskAi()} className="rounded-xl bg-slate-900 border-slate-800" />
                  <Button onClick={() => handleAskAi()} className="rounded-xl"><Bot className="h-4 w-4 mr-1" /> Fragen</Button>
                </div>
              </div>
              <p className="text-[11px] text-slate-400 mt-2">Der KI Assistent liefert Recherche Hinweise. Keine Empfehlung zum Kauf oder Verkauf. Für echte Live Antworten verbinde dein eigenes Modell über die API</p>
            </CardContent>
          </Card>
        </div>

        {/* Watchlist and Ideas */}
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-6 mt-6">
          <Card className="rounded-2xl bg-slate-900 border-slate-800 lg:col-span-2">
            <CardHeader className="pb-2">
              <CardTitle className="text-base flex items-center gap-2"><Star className="h-4 w-4" /> Watchlist & Sektoren</CardTitle>
            </CardHeader>
            <CardContent>
              <Tabs defaultValue="watchlist">
                <TabsList className="rounded-2xl">
                  <TabsTrigger className="rounded-xl" value="watchlist">Watchlist</TabsTrigger>
                  <TabsTrigger className="rounded-xl" value="sectors">Sektoren</TabsTrigger>
                </TabsList>
                <TabsContent value="watchlist">
                  <div className="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-6 gap-2">
                    {watchlist.map((t) => (
                      <button
                        key={t}
                        onClick={() => setTicker(t)}
                        className={`p-3 rounded-xl border ${ticker === t ? "border-sky-500 bg-slate-950" : "border-slate-800 bg-slate-950 hover:border-slate-700"}`}
                      >
                        <div className="text-xs text-slate-400">Ticker</div>
                        <div className="font-semibold">{t}</div>
                      </button>
                    ))}
                  </div>
                </TabsContent>
                <TabsContent value="sectors">
                  <div className="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-3">
                    {[
                      { name: "Tech", change: +((Math.random() - 0.3) * 3).toFixed(2) },
                      { name: "Gesundheit", change: +((Math.random() - 0.3) * 3).toFixed(2) },
                      { name: "Finanzen", change: +((Math.random() - 0.3) * 3).toFixed(2) },
                      { name: "Industrie", change: +((Math.random() - 0.3) * 3).toFixed(2) },
                      { name: "Energie", change: +((Math.random() - 0.3) * 3).toFixed(2) },
                      { name: "Konsum", change: +((Math.random() - 0.3) * 3).toFixed(2) },
                    ].map((s) => (
                      <div key={s.name} className="p-4 rounded-2xl bg-slate-950 border border-slate-800">
                        <div className="text-sm text-slate-400">{s.name}</div>
                        <div className="flex items-center gap-2 mt-1">
                          {s.change >= 0 ? <ArrowUpRight className="h-4 w-4 text-green-400" /> : <ArrowDownRight className="h-4 w-4 text-rose-400" />}
                          <div className="font-semibold">{fmt.format(s.change)}%</div>
                        </div>
                      </div>
                    ))}
                  </div>
                </TabsContent>
              </Tabs>
            </CardContent>
          </Card>

          <Card className="rounded-2xl bg-slate-900 border-slate-800">
            <CardHeader className="pb-2">
              <CardTitle className="text-base flex items-center gap-2"><TrendingUp className="h-4 w-4" /> Ideen Generator</CardTitle>
            </CardHeader>
            <CardContent className="space-y-3">
              <p className="text-sm text-slate-300">Generiert neutrale Ideen aus Kursverlauf und MA14</p>
              <IdeaRow label="Momentum" value={last > series.at(-1).ma ? "positiv" : "negativ"} />
              <IdeaRow label="Trend vs MA" value={last >= series.at(-1).ma ? "+ über MA" : "- unter MA"} />
              <IdeaRow label="6M Drift Demo" value={(fc.at(-1)?.forecast ?? 0) > last ? "aufwärts" : "abwärts"} />
              <Separator className="bg-slate-800" />
              <p className="text-[11px] text-slate-400">Nur zu Bildungszwecken. Keine Empfehlung. Ergänze Fundamentaldaten Bewertung und Risiko</p>
            </CardContent>
          </Card>
        </div>

        {/* Footer */}
        <div className="mt-8 text-center text-xs text-slate-500">
          <p>© {new Date().getFullYear()} Aktien KI Dashboard · Demo Oberfläche</p>
        </div>
      </div>
    </div>
  );
}

Funktion StatCard ({Titel, Wert, Trend }) {
  const up = Trend >= 0;
  return (
    <Card className="rounded-2xl bg-slate-900 border-slate-800">
      <CardHeader className="pb-2">
        <CardTitle className="text-base">{title}</CardTitle>
      </CardHeader>
      <CardContent>
        <div className="flex items-end justify-between">
          <div>
            <div className="text-2xl font-bold">{value}</div>
            <div className={"text-xs mt-1 flex items-center gap-1 " + (up ? "text-green-400" : "text-rose-400") }>
              {up ? <ArrowUpRight className="h-4 w-4" /> : <ArrowDownRight className="h-4 w-4" />}
              {fmt.format(trend)}
            </div>
          </div>
        </div>
      </CardContent>
    </Card>
  );
}

function IdeaRow({ label, value }) {
  return (
    <div className="flex items-center justify-between text-sm">
      <div className="text-slate-400">{label}</div>
      <div className="font-medium">{value}</div>
    </div>
  );
}
