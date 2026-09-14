# Wevlo Base App Template — এই ফোল্ডারটা কী

এইটা Wevlo অ্যাপ **না**। এইটা একটা আলাদা, খুবই ছোট Android প্রজেক্ট — শুধু একটা
WebView যেটা `assets/www/index.html` লোড করে। Wevlo-র on-device APK বিল্ডার
(`ApkTemplateBuilder.java`, মূল প্রজেক্টে) এই APK-টাকেই "ভিত্তি" (base) হিসেবে
নিয়ে, প্রতিটা ইউজারের প্রজেক্টের জন্য package name / app name / HTML-CSS-JS
বদলে নতুন একটা independent APK বানায় — পুরোটাই ফোনের ভেতরে, কোনো সার্ভার বা
Android SDK ছাড়া।

## কেন লাগবে

আমি (Claude) sandbox-এ Android SDK / Google-এর সার্ভারে অ্যাক্সেস নেই, তাই
নিজে থেকে একটা রিয়েল, কম্পাইল করা APK বানাতে পারিনি। কিন্তু এই পুরো প্রজেক্টের
সোর্স কোড লিখে দিয়েছি — আপনাকে এটা **একবার মাত্র** Android Studio দিয়ে বিল্ড
করে নিতে হবে। এরপর যতগুলো ইচ্ছা APK Wevlo দিয়ে বানানো যাবে, প্রতিবারই এই বেস
থেকেই বানাবে, আর কখনো Android Studio লাগবে না — সবই ফোনের মধ্যে।

## Update: base.apk এখন এখানে অলরেডি বসানো আছে

Claude পরে GitHub-এর **Apktool** (বান্ডলড `aapt2` + smali/dexlib2) ব্যবহার করে
sandbox-এর ভেতরেই একটা রিয়েল `base.apk` কম্পাইল করে ফেলেছে (Android Studio
ছাড়াই), এবং সেটা `app/src/main/assets/template/base.apk`-তে বসানো আছে —
তাই **নিচের ধাপগুলো আর করতে হবে না**, যদি না আপনি base app-এর ডিজাইন/কোড
বদলাতে চান।

যদি বদলাতে চান (নতুন WebView ফিচার, ভিন্ন থিম, ইত্যাদি), তাহলে নিচের ধাপগুলো
অনুসরণ করে নতুন করে বিল্ড করুন এবং `base.apk` রিপ্লেস করুন।

---

## ধাপে ধাপে

1. **Android Studio-তে এই ফোল্ডারটা (`BASE_APP_TEMPLATE/`) আলাদা প্রজেক্ট
   হিসেবে ওপেন করুন** (`File > Open`, পুরো Wevlo প্রজেক্ট না, শুধু এই সাব-ফোল্ডার)।

2. Gradle sync শেষ হওয়ার পর, মেনু থেকে:
   `Build > Build Bundle(s) / APK(s) > Build APK(s)`
   (অথবা টার্মিনালে: `./gradlew assembleRelease`)

3. বিল্ড শেষ হলে APK ফাইলটা পাবেন এখানে:
   `app/build/outputs/apk/release/app-release.apk`
   (release না হয়ে debug হলেও চলবে — সিগনেচার যাই হোক, Wevlo সেটা পরে নিজের
   কী দিয়ে আবার sign করে দেয়, তাই এখানে যে সিগনেচার আছে সেটা গুরুত্বপূর্ণ না।)

4. এই ফাইলটাকে কপি করে মূল Wevlo প্রজেক্টে এই path-এ রাখুন:
   ```
   app/src/main/assets/template/base.apk
   ```
   (`template` নামে একটা ফোল্ডার নতুন করে বানাতে হবে `assets` এর ভেতরে।)

5. Wevlo অ্যাপটা আবার বিল্ড/রান করুন। ব্যাস — এখন থেকে "Build APK" →
   "Install APK" বাটনে ক্লিক করলে সত্যিকারের, আলাদা package name-এর, আলাদা
   করে sign করা একটা APK বানিয়ে ইনস্টল করতে দেবে। "Do you want to update
   Wevlo?" ডায়ালগ আর আসবে না, কারণ এখন প্যাকেজ নেম আর সিগনেচার — দুটোই আলাদা।

## এই ফোল্ডারের ফাইলগুলো কী করে (না বদলালেও চলবে, জানার জন্য)

- **`app/src/main/AndroidManifest.xml`** — এখানে ইচ্ছাকৃতভাবে অনেক লম্বা লম্বা
  placeholder বসানো আছে (package name, app label, version name-এর জায়গায়)।
  Wevlo-র `AxmlStringPatcher.java` কম্পাইল হওয়া manifest-এর বাইনারি ভেতরে
  ঢুকে এই placeholder স্ট্রিংগুলো খুঁজে বের করে, আসল ছোট মানগুলো দিয়ে
  in-place বসিয়ে দেয় (লম্বা জায়গার ভেতরে ছোট মান বসানো — কখনো উল্টোটা না,
  এইজন্যই placeholder এত লম্বা রাখা হয়েছে)। **এই তিনটা placeholder স্ট্রিং
  এখানে আর `ApkTemplateBuilder.java`-তে হুবহু (character-by-character) মিলতে
  হবে** — একটায় বদলালে অন্যটায়ও বদলাতে হবে, নাহলে বিল্ড fail করবে।

- **`app/src/main/java/com/wevlo/tpl/base/MainActivity.java`** — পুরো "অ্যাপ"টাই
  এটা: একটা ফুল-স্ক্রিন WebView যেটা `file:///android_asset/www/index.html`
  লোড করে। এর ক্লাস নেইম manifest-এ পুরো (fully-qualified) লেখা আছে
  (শুধু `.MainActivity` না) — এইজন্য যাতে package name বদলে গেলেও Android ঠিক
  ক্লাসটাই খুঁজে পায় (এইটা normal Android behavior — `applicationId` আর
  আসল Java package আলাদা জিনিস, বিস্তারিত manifest-এর ভেতরের কমেন্টে আছে)।

- **`app/src/main/assets/www/index.html`** — শুধু একটা placeholder পেজ, যেটা
  এই base APK-টা একা ইনস্টল করলে দেখাবে। আসল প্রজেক্টে Wevlo এই পুরো
  `assets/www/` ফোল্ডারটাই বদলে দেয় ইউজারের নিজের `index.html` / `style.css`
  / `script.js` দিয়ে।

- **`app/src/main/res/mipmap-*/`** — ডিফল্ট আইকন। ইউজার নিজের আইকন বেছে নিলে
  Wevlo এই PNG ফাইলগুলোই বদলে দেয় (সরাসরি zip entry replace, কোনো resource
  ফরম্যাট নিয়ে ঘাঁটাঘাঁটি লাগে না)।

## আপডেট (২০২৬-০৯-১৪): প্রতিটা toggle-এর *runtime* কোডও এখন try/catch দিয়ে মোড়ানো

আগের ভার্সনে শুধু প্রতিটা toggle-এর **setup** কোড (`onCreate()`-এর ভেতরের
কলগুলো) try/catch দিয়ে মোড়ানো ছিল। কিন্তু কিছু toggle-এর আসল কাজ setup-এর
পরে, পরবর্তীতে একটা callback/listener-এর ভেতরে চলে — আর সেই callback-এর
*body*-টা আগে unguarded ছিল। সবচেয়ে স্পষ্ট উদাহরণ: **Pull to Refresh**-এর
`OnTouchListener` — `setupSwipeRefresh()`-কে try/catch দিয়ে মোড়ানো ছিল ঠিকই,
কিন্তু সেটা শুধু *listener রেজিস্টার করা*টাকে প্রোটেক্ট করে, ইউজার আসলে
টেনে (drag করে) refresh trigger করার সময় যেই কোড রান হয় সেটাকে না। ওই
কোডে কোনো exception (কোনো ডিভাইসে `getScrollY()`/`reload()`-এর অদ্ভুত
আচরণ, ইত্যাদি) ছুঁড়লে সেটা সরাসরি Android-এর touch-dispatch থেকে বেরিয়ে
পুরো অ্যাপ ক্র্যাশ করে দিতে পারত — ঠিক "toggle অন করলেই ক্র্যাশ" (মানে
আসলে ব্যবহার করলে ক্র্যাশ) এই উপসর্গের সাথে মিলে যায়।

এবার নিচের প্রতিটা জায়গায় আলাদা try/catch যোগ করা হয়েছে (শুধু setup না,
runtime callback-ও):
- Pull-to-refresh-এর `OnTouchListener` বডি (সবচেয়ে গুরুত্বপূর্ণ ফিক্স)
  এবং `triggerPullToRefresh()`
- `onPageFinished`, `onReceivedTitle`, `onPermissionRequest` /
  `handleWebPermissionRequest` (এই তিনটা runOnUiThread lambda-সহ)
- Long-press মেনুর প্রতিটা আইটেমের ক্লিক action (`onLongPressMenuClick`)
  আর `copyToClipboard`
- `hideLoadingOverlay()` (animation fail করলেও overlay যেন আটকে না থাকে,
  তার জন্য instant-hide fallback)
- `onDestroy()`-এর `webView.destroy()`
- `onCreate()`-এর ভেতরে `buildTopBar()` আর `buildLoadingOverlay()` কল দুটো
  আলাদাভাবে try/catch দিয়ে মোড়ানো হয়েছে (আগে এই দুটো কল কোনো গার্ড ছাড়াই
  `setContentView()`-এর আগে চলত — `buildLoadingOverlay()` ভেতরে
  `getApplicationInfo().icon` ব্যবহার করে, যেটা কোনো malformed/patched
  icon resource-এ `Resources.NotFoundException` ছুঁড়তে পারে)

এখন যেকোনো একটা toggle-এর কোনো অংশ (setup বা ব্যবহারকালীন, দুটোই) কোনো
exception ছুঁড়লে শুধু সেই একটা ফিচারই নিঃশব্দে স্কিপ হবে (logcat-এ
`MainActivity` ট্যাগে একটা warning সহ) — পুরো অ্যাপ আর ক্র্যাশ করবে না।

**তবু ১০০% গ্যারান্টি না:** এটা এখনো একটা defensive/safety-net ফিক্স —
root cause (ঠিক কোন লাইনে exception হচ্ছিল) logcat ছাড়া নিশ্চিত করা যায়নি।
সোর্সের এই আপডেট build করে `assets/template/base.apk` রিপ্লেস করার পর
সাতটা toggle একে একে রিয়েল ডিভাইসে টেস্ট করুন — এখনো কোনোটা ক্র্যাশ করলে,
এখন logcat-এ ঠিক কোন toggle-এর warning এসেছে সেটাই সরাসরি জায়গাটা বলে দেবে।

## এখনো যা automatic না (future work)

- **App icon টা এখন পর্যন্ত automatic** (ইউজার আইকন বেছে নিলে বসে যায়)।
- **versionCode / versionName প্যাচ করা এখন কাজ করে** placeholder সিস্টেমের
  মাধ্যমে।
- **"Website URL" মোড** (সরাসরি কোনো লাইভ ওয়েবসাইট লোড করা, লোকাল HTML না)
  আপাতত একটা ছোট redirect পেজ বানিয়ে কাজ চালাচ্ছে (`<meta http-equiv=refresh>`)।
  চাইলে পরে সরাসরি `WebView.loadUrl()`-এ আসল URL বসিয়ে দেওয়ার জন্য আরেকটা
  placeholder (যেমন একটা লম্বা "start URL" স্ট্রিং `assets/www/` এর কোনো
  config ফাইলে) যোগ করা যায় — এটা প্রায় একই কৌশলে করা সম্ভব, কিন্তু এখনকার
  redirect-পেজ পদ্ধতিও পুরোপুরি কাজ করে।
