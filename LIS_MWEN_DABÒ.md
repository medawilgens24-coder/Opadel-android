# OPADEL — Pakaj Android (Capacitor) — Odit + Etap Build

## 1) ODIT — SA MWEN JWENN NAN KÒD LA

J'ai analysé `OPADEL_5eF_Gestion_Academique-isolation-3.html` an antye avan mwen touche anyen.
Rezilta a:

### ✅ Sa ki DEJA konpatib ak Android/Capacitor san okenn chanjman
- **Firebase Authentication** pase pa `fetch()` HTTPS dirèk sou `identitytoolkit.googleapis.com` /
  `securetoken.googleapis.com` (Identity Toolkit REST). Sa a se senp rekèt HTTPS — li mache menm jan
  nan yon WebView Android tou, depi aparèy la gen INTERNET permission (Capacitor mete l otomatikman).
- **Firestore** chaje via SDK compat (`firebase-app-compat.js`, `firebase-firestore-compat.js`) sou
  CDN `gstatic.com`, tou sou HTTPS. Menm rezònman: fonksyone san chanjman nan WebView Android.
- **Separasyon done pa UID** (`storageKeyFor(uid)`, `authKeyFor(uid)`, `schools/{uid}/data/...`) pa
  depann sou okenn API navigatè ki pa disponib sou Android — `localStorage` ak Firestore fonksyone
  nòmalman nan yon WebView Capacitor. **Mwen pa t touche lojik sa a ditou.**
- `enablePersistence` (Firestore offline / IndexedDB) — IndexedDB sipòte nan WebView Android modèn.
- Pa gen okenn `Content-Security-Policy` ki ta bloke script CDN gstatic.com yo.
- Pa gen `window.open` / `target="_blank"` — pa gen depandans sou yon navigatè ekstèn.
- `<input type="file">` (enpòte sovgad, foto pwofil/kouvèti) — Capacitor Android gen sipò
  `onShowFileChooser` deja entegre nan `BridgeActivity` li — mache san chanjman.

### ⚠️ 1 CHANJMAN MINIMÒM MWEN TE APLIKE (rezon teknik reyèl)
**Pwoblèm** : Bouton **"Exporter les données"** te itilize teknik `Blob` + `<a download>` pou
telechaje fichye JSON lan. Teknik sa a **PA FONKSYONE nan WebView Android** — WebView pa konn jere
atribi `download` sou yon `<a>`; klik la pa fè anyen (okenn erè, okenn fichye — bouton an ta parèt
"kraze" pou itilizatè Android lan san rezon aparan).
**Poukisa li afekte Android** : se yon limitasyon konnen nan Android System WebView (kontrèman ak
Chrome konplè), pa yon bug nan kòd ou a.
**Chanjman mwen fè** : mwen AJOUTE (pa ranplase) yon chemen altènatif — SÈLMAN lè aplikasyon an ap
kouri kòm app natif Android (`window.Capacitor.isNativePlatform()`), li itilize plugin
`@capacitor/filesystem` pou ekri fichye a epi `@capacitor/share` pou louvri meni "pataje/anrejistre".
**Sou navigatè web** (menm HTML fichye a), konpòtman ORIJINAL la (`Blob` + `<a download>`) rete
EGZAKTEMAN menm jan an — okenn pwovaisyon la retire.
**Risk** : ZERO pou vèsyon web la (chemen an pa menm rive la si `Capacitor` pa egziste). Pou Android,
sa ajoute 2 depandans (`@capacitor/filesystem`, `@capacitor/share`) — deja mete nan `package.json`.

### ℹ️ Limitasyon konnen — mwen PA CHANJE anyen, men fòk ou konnen l
- **`window.print()`** (bouton Enprime/Rapports/Palmarès) : Android System WebView **pa deklannche
  otomatikman** bwat dyalòg enprime a jan yon navigatè konplè (Chrome) ta fè l — sa mande yon ti kras
  kòd natif Android (pa yon senp config) pou konekte `WebView.createPrintDocumentAdapter()` ak
  `android.print.PrintManager`. **Mwen pa t ajoute chanjman sa a** paske se yon modifikasyon KOD
  NATIF Android (pa yon senp pakaj), e règ ou yo mande m pa fè chanjman ki pa strikteman nesesè san
  konfimasyon. Si ou vle fonksyon enprime a mache tankou sou web la, di m — mwen ka ajoute yon ti
  plugin Capacitor pèsonalize pou sa apre.
- **Firestore SDK chaje dinamikman sou entènèt** (via CDN) chak fwa app la louvri : sa vle di premye
  louvri app la SOU TELEFÒN MANDE koneksyon entènèt aktif pou senkronizasyon nwaj la disponib (menm
  jan sou web la deja). Otantifikasyon/koneksyon PA depann de sa (li pase pa REST dirèkteman).

## 2) STRIKTI PWOJÈ A

```
opadel-android/
├── package.json              ← depandans Capacitor + plugins
├── capacitor.config.json     ← konfigirasyon Capacitor (appId, appName, webDir)
├── www/
│   └── index.html            ← APLIKASYON AN, san okenn fonksyon retire/chanje
│                                (sèl chanjman: chemen altènatif Android pou "Exporter")
└── android/                  ← ap kreye otomatikman pa `npx cap add android` (etap 3 pi ba)
```

## 3) ETAP EGZAK — soti nan HTML rive nan APK

**Sa ou bezwen enstale AVAN (sou òdinatè w, pa sou telefòn)** :
- [Node.js](https://nodejs.org) vèsyon 18 oswa pi resan + npm
- [Android Studio](https://developer.android.com/studio) (li vin ak Android SDK + Gradle)
- JDK 17 (Android Studio mòdèn enstale l otomatikman)

### Etap 1 — Telechaje/kopye dosye `opadel-android/` sa a sou òdinatè w
Mete tout 3 fichye/dosye yo (`package.json`, `capacitor.config.json`, `www/`) nan menm folder.

### Etap 2 — Enstale depandans yo
```bash
cd opadel-android
npm install
```

### Etap 3 — Ajoute platfòm Android lan (kreye dosye `android/`)
```bash
npx cap add android
```

### Etap 4 — Senkronize kòd web la (`www/`) ak pwojè Android la
Fè sa a **CHAK FWA** ou modifye `www/index.html` :
```bash
npx cap sync android
```

### Etap 5 — (Opsyonèl men rekòmande) Chanje icon ak splash screen
Pa gen icon pèsonalize kounye a — Capacitor ap itilize icon default li. Pou mete pwòp icon OPADEL ou :
1. Prepare yon imaj kare (omwen 1024×1024px, PNG) ak logo/desen OPADEL ou.
2. Enstale zouti a : `npm install @capacitor/assets --save-dev`
3. Mete imaj la nan `opadel-android/assets/icon.png` (ak `assets/splash.png` si ou vle yon splash).
4. Kouri : `npx capacitor-assets generate --android`
5. `npx cap sync android`

### Etap 6 — Louvri pwojè a nan Android Studio epi build APK la
```bash
npx cap open android
```
Sa ap louvri Android Studio otomatikman ak pwojè `android/` la. Nan Android Studio :
1. Tann Gradle fini "sync" (an ba ekran an, premye fwa a ka pran plizyè minit).
2. Meni : **Build → Build Bundle(s) / APK(s) → Build APK(s)**.
3. Lè li fini, yon notifikasyon parèt anba dwat ekran an ak yon lyen **"locate"** — klike sou li pou
   jwenn fichye a. Li ap konsa: `android/app/build/outputs/apk/debug/app-debug.apk`.

*(Pou yon APK "release" siyen pou pibliye sou Play Store, ou ap bezwen kreye yon keystore siyati —
di m si ou rive la epi m ap gide w etap pa etap pou sa.)*

### Etap 7 — Enstale APK la sou telefòn Android ou
**Opsyon A — Dirèkteman soti nan Android Studio :**
Konekte telefòn ou (kab USB, ak "Débogage USB" aktive nan Paramèt Devlopè), klike bouton ▶️ **Run**
nan Android Studio — l ap enstale epi louvri app la dirèkteman sou telefòn ou.

**Opsyon B — Transfere fichye APK la manyèlman :**
1. Kopye `app-debug.apk` la sou telefòn ou (via kab USB, Google Drive, WhatsApp, elatriye).
2. Sou telefòn lan, louvri fichye a — Android ap mande w otorize "enstale app soti nan sous enkoni"
   (Settings → Security → Install unknown apps) yon sèl fwa.
3. Konfime enstalasyon an — app **OPADEL** ap parèt nan lis app telefòn ou ak pwòp icon li.

## 4) VERIFIKASYON FINAL (jan ou te mande)

- [x] Tout fonksyon aplikasyon orijinal la toujou la — **AUKENN fonksyon pa t retire**.
- [x] UI/UX la pa chanje — zewo chanjman sou HTML/CSS/estrikti paj yo.
- [x] Firebase Authentication (REST) — pa touche, mache dirèkteman sou HTTPS.
- [x] Login/Logout/Kreye kont — pa touche.
- [x] Firebase UID rete sèl idantifyan pou separasyon done — pa touche.
- [x] Admin A pa ka wè done Admin B (ak vis vèrsa) — lojik `storageKeyFor`/`authKeyFor`/
      `schools/{uid}/...` pa touche ditou.
- [x] Firestore reads/writes/senkronizasyon online — pa touche.
- [x] Pa depann de `file://` ni `content://` — `capacitor.config.json` konfigire ak
      `androidScheme: "https"` (orijin `https://localhost`), egzakteman jan règ #8 mande a.
- [x] WebView Android ka kominike ak Firebase sou HTTPS — sèl depandans se INTERNET permission,
      ki vini otomatikman ak template Android Capacitor la (pa gen konfigirasyon siplemantè pou fè).
- [ ] Build APK — pa posib pou m fè l isit la (mwen pa gen aksè entènèt/Android SDK nan
      anviwònman sa a) — swiv Etap 1-7 pi wo sou pwòp òdinatè w.
- [ ] Enstale/teste sou yon telefòn Android reyèl — fòk ou fè sa apre build la.

## 5) SI OU BEZWEN M AL PI LWEN
- Fonksyon **Enprime/Rapports** (`window.print()`) : mwen ka ajoute yon ti plugin Capacitor
  pèsonalize pou konekte l ak `PrintManager` Android — di m si ou vle sa.
- **APK siyen pou Play Store** : gide etap-pa-etap pou kreye keystore + `build.gradle` release config.
- **Icon/splash screen** pèsonalize : voye m yon imaj logo, m ka prepare tout gwosè yo ak
  `@capacitor/assets`.
