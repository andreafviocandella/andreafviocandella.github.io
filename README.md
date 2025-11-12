html = r"""<!DOCTYPE html>
<html lang="id">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Kalkulator Body Ideal & Fitness</title>
    <meta name="description" content="Aplikasi kalkulator body ideal & fitness untuk semua usia & gender: BMI, Body Fat, BMR, TDEE, makro harian, hidrasi, heart-rate zones, saran latihan, cetak PDF, dan penyimpanan lokal." />
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://unpkg.com">
    <link rel="preconnect" href="https://cdn.jsdelivr.net">
    <!-- React & Babel -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.development.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.development.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <!-- html2canvas & jsPDF for PDF export -->
    <script src="https://cdn.jsdelivr.net/npm/html2canvas@1.4.1/dist/html2canvas.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/jspdf@2.5.1/dist/jspdf.umd.min.js"></script>
    <style>
      @media print {
        header .print\\:hidden { display: none; }
        header, .print\\:shadow-none { box-shadow: none !important; }
        a { text-decoration: none; }
      }
    </style>
  </head>
  <body class="bg-slate-50 text-slate-800">
    <div id="root"></div>

    <script type="text/babel">
      const { useState, useMemo, useEffect } = React;

      // ------------------ I18N ------------------
      const DICT = {
        id: {
          appTitle: "Kalkulator Body Ideal & Fitness",
          unitMetric: "Metrik",
          unitImperial: "Imperial",
          switchTo: (u) => \`Ganti → \${u}\`,
          profile: "Data Diri",
          age: "Umur (tahun)",
          gender: "Gender",
          male: "Pria",
          female: "Wanita",
          other: "Lainnya / Non-biner",
          height: (u) => \`Tinggi (\${u})\`,
          weight: (u) => \`Berat (\${u})\`,
          neck: (u) => \`Leher (\${u})\`,
          waist: (u) => \`Pinggang (\${u})\`,
          hip: (u) => \`Pinggul (\${u})\`,
          activity: "Aktivitas",
          sedentary: "Sedentary (1–2k langkah)",
          light: "Ringan (2–6k langkah)",
          moderate: "Sedang (6–10k + 1–3x latihan)",
          very: "Sangat aktif (4–6x latihan)",
          athlete: "Atlet / Pekerja fisik",
          goal: "Tujuan",
          cut: "Fat Loss (Cut)",
          recomp: "Recomp (turun lemak/naik otot)",
          maintain: "Maintain",
          bulk: "Muscle Gain (Bulk)",
          level: "Level Pengalaman",
          beginner: "Pemula",
          intermediate: "Menengah",
          advanced: "Mahir",
          measureTips: "Tips pengukuran: leher di bawah jakun; pinggang di titik tersempit/di atas pusar; pinggul di titik terlebar bokong. Ukur saat relaks.",

          summary: "Ringkasan Badan & Kalori",
          heightLabel: "Tinggi",
          weightLabel: "Berat",
          bmi: "BMI",
          bf: "Body Fat %",
          fatMass: "Massa Lemak",
          leanMass: "Massa Bebas Lemak",
          bmr: "BMR",
          tdee: "TDEE",
          targetCal: "Target Kalori",

          macrosHydration: "Makro Harian & Hidrasi",
          protein: "Protein",
          fat: "Lemak",
          carbs: "Karbo",
          water: "Air minum",
          macroNote: "Rasio makro disesuaikan otomatis oleh tujuan. Sesuaikan preferensi & toleransi pribadi.",

          hrZones: "Heart Rate Zones (Perkiraan)",
          mhr: "MHR (220 - umur)",
          z1: "Z1 Recovery",
          z2: "Z2 Easy/Endurance",
          z3: "Z3 Tempo",
          z4: "Z4 Threshold",
          z5: "Z5 VO₂max",

          plan: "Saran Latihan Mingguan (Adaptif)",
          notes: "Catatan & Batasan",
          notesList: [
            "Untuk usia < 20 tahun, kategori BMI memakai persentil usia. Alat ini menampilkan catatan umum, bukan diagnosis.",
            "Estimasi Body Fat (US Navy) sensitif pada akurasi pengukuran.",
            "BMR/TDEE adalah perkiraan. Pantau 2–3 minggu lalu sesuaikan 100–200 kcal.",
            "Jika memiliki kondisi medis/kehamilan, konsultasikan profesional terlebih dahulu.",
          ],

          kidsModeTitle: "Mode Anak/Remaja (<20 th)",
          kidsHelp: "Masukkan perkiraan persentil BMI‑for‑age dari grafik WHO/CDC untuk klasifikasi.",
          percentile: "Persentil BMI (0–100)",
          classify: "Klasifikasikan",
          catUnder: "Underweight (< P5)",
          catHealthy: "Sehat (P5–P85)",
          catOver: "Overweight (P85–P95)",
          catObese: "Obesitas (≥ P95)",
          openCharts: "Buka grafik WHO/CDC",

          export: "Cetak / PDF",
          exportDesc: "Cetak halaman ini atau simpan sebagai PDF.",
          exportBtn: "Unduh PDF",
          saved: "Data otomatis tersimpan.",

          footer: "Edukasi kesehatan & kebugaran.",
        },
        en: {
          appTitle: "Ideal Body & Fitness Calculator",
          unitMetric: "Metric",
          unitImperial: "Imperial",
          switchTo: (u) => \`Switch → \${u}\`,
          profile: "Profile",
          age: "Age (years)",
          gender: "Gender",
          male: "Male",
          female: "Female",
          other: "Other / Non‑binary",
          height: (u) => \`Height (\${u})\`,
          weight: (u) => \`Weight (\${u})\`,
          neck: (u) => \`Neck (\${u})\`,
          waist: (u) => \`Waist (\${u})\`,
          hip: (u) => \`Hip (\${u})\`,
          activity: "Activity",
          sedentary: "Sedentary (1–2k steps)",
          light: "Light (2–6k steps)",
          moderate: "Moderate (6–10k + 1–3x training)",
          very: "Very active (4–6x)",
          athlete: "Athlete / Manual labor",
          goal: "Goal",
          cut: "Fat Loss (Cut)",
          recomp: "Recomp (fat↓ / muscle↑)",
          maintain: "Maintain",
          bulk: "Muscle Gain (Bulk)",
          level: "Experience Level",
          beginner: "Beginner",
          intermediate: "Intermediate",
          advanced: "Advanced",
          measureTips: "Measuring tips: neck below Adam's apple; waist at narrowest/above navel; hip at widest over glutes. Measure relaxed.",

          summary: "Body & Calories Summary",
          heightLabel: "Height",
          weightLabel: "Weight",
          bmi: "BMI",
          bf: "Body Fat %",
          fatMass: "Fat Mass",
          leanMass: "Lean Mass",
          bmr: "BMR",
          tdee: "TDEE",
          targetCal: "Target Calories",

          macrosHydration: "Daily Macros & Hydration",
          protein: "Protein",
          fat: "Fat",
          carbs: "Carbs",
          water: "Water intake",
          macroNote: "Macro ratios auto‑tuned by goal. Adjust to preference & tolerance.",

          hrZones: "Heart Rate Zones (Estimate)",
          mhr: "MHR (220 - age)",
          z1: "Z1 Recovery",
          z2: "Z2 Easy/Endurance",
          z3: "Z3 Tempo",
          z4: "Z4 Threshold",
          z5: "Z5 VO₂max",

          plan: "Weekly Training Advice (Adaptive)",
          notes: "Notes & Limitations",
          notesList: [
            "For <20y, BMI category uses age percentiles. This tool provides general guidance only.",
            "US Navy Body Fat estimate is sensitive to measurement accuracy.",
            "BMR/TDEE are estimates. Track 2–3 weeks then adjust 100–200 kcal.",
            "If you have medical conditions/pregnancy, consult a professional first.",
          ],

          kidsModeTitle: "Kids/Teens Mode (<20y)",
          kidsHelp: "Enter an approximate BMI‑for‑age percentile from WHO/CDC charts to classify.",
          percentile: "BMI Percentile (0–100)",
          classify: "Classify",
          catUnder: "Underweight (< P5)",
          catHealthy: "Healthy (P5–P85)",
          catOver: "Overweight (P85–P95)",
          catObese: "Obesity (≥ P95)",
          openCharts: "Open WHO/CDC charts",

          export: "Print / PDF",
          exportDesc: "Print this page or save to PDF.",
          exportBtn: "Download PDF",
          saved: "Data auto‑saved.",

          footer: "Health & fitness education.",
        },
      };

      function BodyFitnessApp() {
        const [lang, setLang] = useState(localStorage.getItem("bf_lang") || "id");
        const t = DICT[lang];

        const [units, setUnits] = useState(localStorage.getItem("bf_units") || "metric");
        const [age, setAge] = useState(Number(localStorage.getItem("bf_age") || 30));
        const [gender, setGender] = useState(localStorage.getItem("bf_gender") || "male");
        const [height, setHeight] = useState(Number(localStorage.getItem("bf_height") || 170));
        const [weight, setWeight] = useState(Number(localStorage.getItem("bf_weight") || 70));
        const [neck, setNeck] = useState(Number(localStorage.getItem("bf_neck") || 38));
        const [waist, setWaist] = useState(Number(localStorage.getItem("bf_waist") || 80));
        const [hip, setHip] = useState(Number(localStorage.getItem("bf_hip") || 95));
        const [activity, setActivity] = useState(localStorage.getItem("bf_activity") || "moderate");
        const [goal, setGoal] = useState(localStorage.getItem("bf_goal") || "recomp");
        const [experience, setExperience] = useState(localStorage.getItem("bf_exp") || "beginner");

        const [hasPercentile, setHasPercentile] = useState(localStorage.getItem("bf_hasP") === "1");
        const [percentile, setPercentile] = useState(Number(localStorage.getItem("bf_pct") || 50));

        useEffect(() => {
          localStorage.setItem("bf_lang", lang);
          localStorage.setItem("bf_units", units);
          localStorage.setItem("bf_age", String(age));
          localStorage.setItem("bf_gender", gender);
          localStorage.setItem("bf_height", String(height));
          localStorage.setItem("bf_weight", String(weight));
          localStorage.setItem("bf_neck", String(neck));
          localStorage.setItem("bf_waist", String(waist));
          localStorage.setItem("bf_hip", String(hip));
          localStorage.setItem("bf_activity", activity);
          localStorage.setItem("bf_goal", goal);
          localStorage.setItem("bf_exp", experience);
          localStorage.setItem("bf_hasP", hasPercentile ? "1" : "0");
          localStorage.setItem("bf_pct", String(percentile));
        }, [lang, units, age, gender, height, weight, neck, waist, hip, activity, goal, experience, hasPercentile, percentile]);

        const isMetric = units === "metric";
        const round = (n, d = 2) => Math.round(n * 10 ** d) / 10 ** d;
        const toMetricWeight = (w) => (isMetric ? w : w * 0.45359237);
        const toMetricHeight = (h) => (isMetric ? h : h * 2.54);
        const toMetricCirc = (v) => (isMetric ? v : v * 2.54);
        const fromMetricWeight = (kg) => (isMetric ? kg : kg / 0.45359237);
        const fromMetricHeight = (cm) => (isMetric ? cm : cm / 2.54);

        const calcBMI = (wKg, hCm) => wKg / ((hCm / 100) ** 2);
        const bmiCategoryAdult = (bmi) => {
          if (bmi < 18.5) return "Underweight";
          if (bmi < 25) return "Normal";
          if (bmi < 30) return "Overweight";
          return "Obese";
        };

        const calcBMR = (wKg, hCm, ageYears, g) => {
          const s = g === "male" ? 5 : g === "female" ? -161 : -78;
          return 10 * wKg + 6.25 * hCm - 5 * ageYears + s;
        };

        const activityMult = { sedentary: 1.2, light: 1.375, moderate: 1.55, very: 1.725, athlete: 1.9 };

        const calcBodyFat = (hCm, nCm, wstCm, hpCm, g) => {
          const log10 = (x) => Math.log10(x);
          if (g === "male") return 86.01 * log10(wstCm - nCm) - 70.041 * log10(hCm) + 36.76;
          return 163.205 * log10(wstCm + (hpCm ?? 0) - nCm) - 97.684 * log10(hCm) - 78.387;
        };

        const macroSplit = (totalKcal, wKg, goal) => {
          const proteinPerKg = goal === "cut" ? 2.2 : goal === "bulk" ? 1.8 : 2.0;
          const proteinG = proteinPerKg * wKg;
          const fatPct = goal === "cut" ? 0.3 : 0.25;
          const fatG = (totalKcal * fatPct) / 9;
          const carbG = (totalKcal - (proteinG * 4 + fatG * 9)) / 4;
          return { proteinG, fatG, carbG };
        };

        const waterMlPerDay = (wKg, goal) => (goal === "maintain" || goal === "recomp" ? 32 : 35) * wKg;

        const heartRateZones = (age) => {
          const max = 220 - age;
          return {
            max,
            z1: [0.5 * max, 0.6 * max],
            z2: [0.6 * max, 0.7 * max],
            z3: [0.7 * max, 0.8 * max],
            z4: [0.8 * max, 0.9 * max],
            z5: [0.9 * max, 1.0 * max],
          };
        };

        const metrics = useMemo(() => {
          const hCm = toMetricHeight(height);
          const wKg = toMetricWeight(weight);
          const nCm = toMetricCirc(neck);
          const wstCm = toMetricCirc(waist);
          const hpCm = toMetricCirc(hip);

          const bmi = calcBMI(wKg, hCm);
          const bmiCat = age >= 20 ? bmiCategoryAdult(bmi) : "Percentil usia (lihat grafik)";
          const bmr = calcBMR(wKg, hCm, age, gender);
          const tdee = bmr * (activityMult[activity] ?? 1.55);

          const bf = calcBodyFat(hCm, nCm, wstCm, gender === "female" ? hpCm : null, gender);
          const fatMass = (bf / 100) * wKg;
          const leanMass = wKg - fatMass;

          let targetKcal = tdee;
          if (goal === "cut") targetKcal = tdee * 0.8;
          if (goal === "bulk") targetKcal = tdee * 1.1;
          if (goal === "recomp") targetKcal = tdee * 0.95;

          const macros = macroSplit(targetKcal, wKg, goal);
          const water = waterMlPerDay(wKg, goal);
          const hr = heartRateZones(age);

          return { hCm, wKg, bmi, bmiCat, bmr, tdee, bf, fatMass, leanMass, targetKcal, macros, water, hr };
        }, [units, age, gender, height, weight, neck, waist, hip, activity, goal]);

        const kidsCategory = useMemo(() => {
          if (age >= 20) return null;
          if (!hasPercentile) return null;
          const p = percentile;
          if (p < 5) return lang === "id" ? DICT.id.catUnder : DICT.en.catUnder;
          if (p < 85) return lang === "id" ? DICT.id.catHealthy : DICT.en.catHealthy;
          if (p < 95) return lang === "id" ? DICT.id.catOver : DICT.en.catOver;
          return lang === "id" ? DICT.id.catObese : DICT.en.catObese;
        }, [age, hasPercentile, percentile, lang]);

        async function exportPDF() {
          try {
            const node = document.getElementById("bf-root");
            if (!node) throw new Error("No node");
            const canvas = await html2canvas(node, { scale: 2, useCORS: true });
            const imgData = canvas.toDataURL("image/png");
            const { jsPDF } = window.jspdf;
            const pdf = new jsPDF("p", "mm", "a4");
            const pageWidth = pdf.internal.pageSize.getWidth();
            const pageHeight = pdf.internal.pageSize.getHeight();
            const ratio = canvas.width / canvas.height;
            const w = pageWidth;
            const h = w / ratio;
            let y = 0;
            if (h <= pageHeight) {
              pdf.addImage(imgData, "PNG", 0, 0, w, h);
            } else {
              let sH = 0;
              while (sH < canvas.height) {
                const slice = document.createElement("canvas");
                const sCtx = slice.getContext("2d");
                const sliceHeight = Math.min(canvas.height - sH, (pageHeight * canvas.width) / pageWidth);
                slice.width = canvas.width;
                slice.height = sliceHeight;
                sCtx.drawImage(canvas, 0, sH, canvas.width, sliceHeight, 0, 0, canvas.width, sliceHeight);
                const img = slice.toDataURL("image/png");
                if (y > 0) pdf.addPage();
                pdf.addImage(img, "PNG", 0, 0, w, (w * sliceHeight) / canvas.width);
                sH += sliceHeight;
                y += pageHeight;
              }
            }
            pdf.save(\`BodyFitness_\${new Date().toISOString().slice(0,10)}.pdf\");
          } catch (e) {
            window.print();
          }
        }

        const labelUnitWeight = isMetric ? "kg" : "lbs";
        const labelUnitHeight = isMetric ? "cm" : "in";
        const labelUnitCirc = labelUnitHeight;

        return (
          <div id="bf-root" className="min-h-screen w-full bg-slate-50 text-slate-800 print:bg-white">
            <header className="sticky top-0 z-10 bg-white/80 backdrop-blur border-b border-slate-200 print:static print:border-none">
              <div className="mx-auto max-w-6xl px-4 py-4 flex items-center justify-between gap-3">
                <h1 className="text-2xl md:text-3xl font-semibold">{t.appTitle}</h1>
                <div className="flex items-center gap-2">
                  <button className={'px-3 py-1 rounded-full text-sm ' + (units === 'metric' ? 'bg-slate-900 text-white' : 'bg-slate-200')} onClick={() => setUnits('metric')}>{t.unitMetric}</button>
                  <button className={'px-3 py-1 rounded-full text-sm ' + (units === 'imperial' ? 'bg-slate-900 text-white' : 'bg-slate-200')} onClick={() => setUnits('imperial')}>{t.unitImperial}</button>
                  <select value={lang} onChange={(e) => setLang(e.target.value)} className="px-3 py-1 rounded-full text-sm border border-slate-300 bg-white hover:bg-slate-50">
                    <option value="id">Bahasa</option>
                    <option value="en">English</option>
                  </select>
                  <button className="px-3 py-1 rounded-full text-sm border border-slate-300 bg-white hover:bg-slate-100 print:hidden" onClick={exportPDF} title={t.export}>
                    {t.exportBtn}
                  </button>
                </div>
              </div>
            </header>

            <main className="mx-auto max-w-6xl px-4 py-6 grid grid-cols-1 lg:grid-cols-3 gap-6">
              <section className="lg:col-span-1">
                <div className="bg-white rounded-2xl shadow p-4 md:p-6 space-y-4 print:shadow-none print:border">
                  <h2 className="text-xl font-semibold">{t.profile}</h2>
                  <div className="grid grid-cols-2 gap-3">
                    <LabeledInput label={t.age} type="number" value={age} onChange={setAge} min={1} max={100} />
                    <div className="flex flex-col">
                      <label className="text-sm text-slate-600">{t.gender}</label>
                      <select value={gender} onChange={(e) => setGender(e.target.value)} className="mt-1 rounded-xl border border-slate-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-slate-400">
                        <option value="male">{t.male}</option>
                        <option value="female">{t.female}</option>
                        <option value="other">{t.other}</option>
                      </select>
                    </div>
                    <LabeledInput label={t.height(labelUnitHeight)} type="number" value={height} onChange={setHeight} min={isMetric ? 80 : 30} max={isMetric ? 230 : 90} />
                    <LabeledInput label={t.weight(labelUnitWeight)} type="number" value={weight} onChange={setWeight} min={isMetric ? 20 : 45} max={isMetric ? 250 : 550} />
                  </div>

                  <h3 className="text-lg font-medium pt-2">Lingkar & Body Fat</h3>
                  <div className="grid grid-cols-3 gap-3">
                    <LabeledInput label={t.neck(labelUnitCirc)} type="number" value={neck} onChange={setNeck} min={isMetric ? 20 : 8} max={isMetric ? 60 : 24} />
                    <LabeledInput label={t.waist(labelUnitCirc)} type="number" value={waist} onChange={setWaist} min={isMetric ? 40 : 16} max={isMetric ? 160 : 63} />
                    <LabeledInput label={t.hip(labelUnitCirc)} type="number" value={hip} onChange={setHip} min={isMetric ? 50 : 20} max={isMetric ? 180 : 71} />
                  </div>

                  <div className="grid grid-cols-2 gap-3 pt-2">
                    <div className="flex flex-col">
                      <label className="text-sm text-slate-600">{t.activity}</label>
                      <select value={activity} onChange={(e) => setActivity(e.target.value)} className="mt-1 rounded-xl border border-slate-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-slate-400">
                        <option value="sedentary">{t.sedentary}</option>
                        <option value="light">{t.light}</option>
                        <option value="moderate">{t.moderate}</option>
                        <option value="very">{t.very}</option>
                        <option value="athlete">{t.athlete}</option>
                      </select>
                    </div>
                    <div className="flex flex-col">
                      <label className="text-sm text-slate-600">{t.goal}</label>
                      <select value={goal} onChange={(e) => setGoal(e.target.value)} className="mt-1 rounded-xl border border-slate-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-slate-400">
                        <option value="cut">{t.cut}</option>
                        <option value="recomp">{t.recomp}</option>
                        <option value="maintain">{t.maintain}</option>
                        <option value="bulk">{t.bulk}</option>
                      </select>
                    </div>
                  </div>

                  <div className="flex flex-col">
                    <label className="text-sm text-slate-600">{t.level}</label>
                    <select value={experience} onChange={(e) => setExperience(e.target.value)} className="mt-1 rounded-xl border border-slate-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-slate-400">
                      <option value="beginner">{t.beginner}</option>
                      <option value="intermediate">{t.intermediate}</option>
                      <option value="advanced">{t.advanced}</option>
                    </select>
                  </div>

                  <InfoBox>
                    <p className="text-sm">{t.measureTips}</p>
                    {age < 20 && (
                      <div className="mt-3">
                        <h4 className="font-medium">{t.kidsModeTitle}</h4>
                        <p className="text-sm text-slate-600">{t.kidsHelp}</p>
                        <div className="mt-2 flex items-center gap-3">
                          <label className="flex items-center gap-2 text-sm">
                            <input type="checkbox" checked={hasPercentile} onChange={(e) => setHasPercentile(e.target.checked)} />
                            <span>Enable</span>
                          </label>
                          <input type="number" min={0} max={100} value={percentile} onChange={(e) => setPercentile(Number(e.target.value))} className="w-28 rounded-xl border border-slate-300 px-3 py-2" placeholder={t.percentile} />
                          <a href="https://www.cdc.gov/growthcharts/clinical_charts.htm" target="_blank" rel="noreferrer" className="text-blue-600 underline text-sm">{t.openCharts}</a>
                        </div>
                        {hasPercentile && (
                          <div className="text-sm mt-2">
                            <strong>{t.classify}:</strong> {kidsCategory || "—"}
                          </div>
                        )}
                      </div>
                    )}
                  </InfoBox>

                  <InfoBox>
                    <p className="text-xs text-emerald-700">{t.saved}</p>
                  </InfoBox>

                  <InfoBox>
                    <h4 className="font-medium mb-1">{t.export}</h4>
                    <p className="text-sm text-slate-600 mb-2">{t.exportDesc}</p>
                    <button onClick={exportPDF} className="px-3 py-2 rounded-xl border border-slate-300 bg-white hover:bg-slate-100">{t.exportBtn}</button>
                  </InfoBox>
                </div>
              </section>

              <section className="lg:col-span-2">
                <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
                  <Card title={t.summary}>
                    <div className="grid grid-cols-2 gap-x-4 gap-y-2 text-sm">
                      <Row label={t.heightLabel} value={\`\${round(fromMetricHeight(metrics.hCm))} \${labelUnitHeight}\`} />
                      <Row label={t.weightLabel} value={\`\${round(fromMetricWeight(metrics.wKg))} \${labelUnitWeight}\`} />
                      <Row label={t.bmi} value={\`\${round(metrics.bmi)} (\${metrics.bmiCat})\`} />
                      <Row label={t.bf} value={\`\${round(metrics.bf)}%\`} />
                      <Row label={t.fatMass} value={\`\${round(metrics.fatMass)} kg\`} />
                      <Row label={t.leanMass} value={\`\${round(metrics.leanMass)} kg\`} />
                      <Row label={t.bmr} value={\`\${Math.round(metrics.bmr)} kcal\`} />
                      <Row label={t.tdee} value={\`\${Math.round(metrics.tdee)} kcal\`} />
                      <Row label={t.targetCal} value={\`\${Math.round(metrics.targetKcal)} kcal (\${goalLabel(goal, lang)})\`} />
                    </div>
                  </Card>

                  <Card title={t.macrosHydration}>
                    <div className="grid grid-cols-2 gap-x-4 gap-y-2 text-sm">
                      <Row label={t.protein} value={\`\${Math.round(metrics.macros.proteinG)} g\`} />
                      <Row label={t.fat} value={\`\${Math.round(metrics.macros.fatG)} g\`} />
                      <Row label={t.carbs} value={\`\${Math.round(metrics.macros.carbG)} g\`} />
                      <Row label={t.water} value={\`\${Math.round(metrics.water)} ml (~\${round(metrics.water/1000)} L)\`} />
                      <div className="col-span-2 text-xs text-slate-500">{t.macroNote}</div>
                    </div>
                  </Card>

                  <Card title={t.hrZones}>
                    <div className="text-sm space-y-1">
                      <div>{t.mhr}: <strong>{metrics.hr.max} bpm</strong></div>
                      <Zone label={t.z1} range={metrics.hr.z1} />
                      <Zone label={t.z2} range={metrics.hr.z2} />
                      <Zone label={t.z3} range={metrics.hr.z3} />
                      <Zone label={t.z4} range={metrics.hr.z4} />
                      <Zone label={t.z5} range={metrics.hr.z5} />
                      <div className="text-xs text-slate-500 pt-1">Use HR monitor for accuracy / gunakan monitor detak untuk akurasi.</div>
                    </div>
                  </Card>

                  <Card title={t.plan}>
                    <ul className="list-disc pl-5 space-y-1 text-sm">
                      {planAdvice(age, goal, experience, lang).map((a, i) => (
                        <li key={i}>{a}</li>
                      ))}
                    </ul>
                  </Card>

                  <Card title={t.notes}>
                    <ul className="list-disc pl-5 space-y-1 text-sm">
                      {t.notesList.map((n, i) => (<li key={i}>{n}</li>))}
                    </ul>
                  </Card>
                </div>
              </section>
            </main>

            <footer className="mx-auto max-w-6xl px-4 pb-10 text-xs text-slate-500">
              © {new Date().getFullYear()} Body Fitness App — {t.footer}
            </footer>
          </div>
        );
      }

      function Card({ title, children }) {
        return (
          <div className="bg-white rounded-2xl shadow p-4 md:p-6 print:shadow-none print:border">
            <h3 className="text-lg font-semibold mb-3">{title}</h3>
            {children}
          </div>
        );
      }

      function InfoBox({ children }) {
        return <div className="bg-slate-50 border border-slate-200 rounded-xl p-3">{children}</div>;
      }

      function Row({ label, value }) {
        return (
          <div className="flex items-center justify-between gap-3">
            <div className="text-slate-500">{label}</div>
            <div className="font-medium">{value}</div>
          </div>
        );
      }

      function Zone({ label, range }) {
        const [a, b] = range;
        return (
          <div className="flex items-center justify-between border-b last:border-b-0 py-1">
            <div className="text-slate-600 text-sm">{label}</div>
            <div className="font-medium text-sm">{Math.round(a)}–{Math.round(b)} bpm</div>
          </div>
        );
      }

      function LabeledInput({ label, type, value, onChange, min, max }) {
        return (
          <div className="flex flex-col">
            <label className="text-sm text-slate-600">{label}</label>
            <input type={type} min={min} max={max} value={value}
              onChange={(e) => onChange(Number(e.target.value))}
              className="mt-1 rounded-xl border border-slate-300 px-3 py-2 focus:outline-none focus:ring-2 focus:ring-slate-400" />
          </div>
        );
      }

      function goalLabel(g, lang) {
        if (lang === "en") {
          if (g === "cut") return "Deficit for Fat Loss";
          if (g === "bulk") return "Surplus for Muscle Gain";
          if (g === "recomp") return "Recomposition (light deficit)";
          return "Maintenance";
        }
        if (g === "cut") return "Defisit untuk Fat Loss";
        if (g === "bulk") return "Surplus untuk Muscle Gain";
        if (g === "recomp") return "Rekomposisi (defisit ringan)";
        return "Pemeliharaan";
      }

      function planAdvice(age, goal, exp, lang) {
        const L = lang === "id" ? {
          kid1: "Fokus aktivitas menyenangkan: lari kecil, bersepeda, berenang, permainan luar ruang 60 menit/hari.",
          kid2: "Hindari beban berat; koordinasi & mobilitas jadi prioritas.",
          teen1: "3–5x/minggu gabungan aerobik + kekuatan dasar (beban tubuh/mesin ringan).",
          teen2: "Tekankan teknik & keselamatan. Tidur cukup, protein memadai.",
          senior1: "Latihan low‑impact 3–5x/minggu: jalan cepat, sepeda statis, berenang.",
          senior2: "2–3x/minggu kekuatan beban sedang + latihan keseimbangan & mobilitas.",
          adult1: "Kardio 2–4 sesi/minggu (20–45 mnt) campur steady & interval sesuai zona HR.",
          adult2: "Kekuatan 2–4 sesi/minggu (upper/lower atau push‑pull‑legs). Progressive overload.",
          cut: "Prioritaskan defisit kalori, protein tinggi, langkah 8–12k.",
          bulk: "Surplus moderat, fokus compound lifts, tidur 7–9 jam.",
          recomp: "Defisit ringan + latihan kekuatan konsisten + kardio ringan‑sedang.",
          maintain: "Jaga volume latihan sedang dan gaya hidup aktif.",
          beg: "Mulai 2–3 hari kekuatan + 2 hari kardio. Kuasai form dulu.",
          int: "3–4 hari kekuatan terstruktur + 2 hari kardio/HIIT moderat.",
          adv: "Periodisasi 4–6 minggu, atur volume & intensitas, jadwalkan deload.",
        } : {
          kid1: "Prioritize fun activities: easy runs, cycling, swimming, outdoor play 60 min/day.",
          kid2: "Avoid heavy loads; focus on coordination & mobility.",
          teen1: "3–5x/week mix aerobic + basic strength (bodyweight/light machines).",
          teen2: "Emphasize technique & safety. Sleep well and get enough protein.",
          senior1: "Low‑impact training 3–5x/week: brisk walk, stationary bike, swim.",
          senior2: "2–3x/week moderate strength + balance & mobility work.",
          adult1: "Cardio 2–4 sessions/week (20–45 min) mix steady & intervals per HR zones.",
          adult2: "Strength 2–4 sessions/week (upper/lower or push‑pull‑legs). Progressive overload.",
          cut: "Calorie deficit focus, high protein, 8–12k daily steps.",
          bulk: "Moderate surplus, compound lifts, 7–9h sleep.",
          recomp: "Light deficit + consistent lifting + light‑moderate cardio.",
          maintain: "Maintain moderate volume & active habits.",
          beg: "Start 2–3 days strength + 2 days cardio. Master form first.",
          int: "3–4 days structured strength + 2 days cardio/HIIT.",
          adv: "Periodize (4–6wk mesos), manage volume/intensity, planned deloads.",
        };

        const base = [];
        const isKid = age < 14;
        const isTeen = age >= 14 && age < 18;
        const isSenior = age >= 60;
        if (isKid) { base.push(L.kid1, L.kid2); }
        else if (isTeen) { base.push(L.teen1, L.teen2); }
        else if (isSenior) { base.push(L.senior1, L.senior2); }
        else { base.push(L.adult1, L.adult2); }

        if (goal === "cut") base.push(L.cut);
        if (goal === "bulk") base.push(L.bulk);
        if (goal === "recomp") base.push(L.recomp);
        if (goal === "maintain") base.push(L.maintain);

        if (exp === "beginner") base.push(L.beg);
        if (exp === "intermediate") base.push(L.int);
        if (exp === "advanced") base.push(L.adv);

        return base;
      }

      const root = ReactDOM.createRoot(document.getElementById('root'));
      root.render(<BodyFitnessApp />);
    </script>
  </body>
</html>
"""
with open('/mnt/data/body_fitness_calculator.html', 'w', encoding='utf-8') as f:
    f.write(html)
print("Saved to /mnt/data/body_fitness_calculator.html")
