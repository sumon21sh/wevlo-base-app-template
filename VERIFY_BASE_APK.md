# base.apk ক্লিন কিনা কীভাবে যাচাই করবেন

## কেন এই ফাইলটা আছে
আগে একবার `assets/template/base.apk`-এর ভেতরে একটা সম্পূর্ণ অপ্রাসঙ্গিক
লাইব্রেরির (`io/github/mohammedbaqernull/logger/logwire`) চিহ্ন পাওয়া
গিয়েছিল — যদিও `BASE_APP_TEMPLATE` কোনো dependency ব্যবহারই করে না। মানে
সেই সময়ের build pipeline-টা dirty ছিল, আর সেটা থেকে বানানো APK বিশ্বাস
করা যায়নি। এই চেকলিস্টটা যেকোনো নতুন base.apk বসানোর আগে চালান, যেন এই
সমস্যাটা আবার না ঘটে।

## যেভাবে বিল্ড করবেন (প্রস্তাবিত: GitHub Actions)
`.github/workflows/build-base-apk.yml` এই রিপোতে যোগ করা আছে। এটা প্রতিবার
সম্পূর্ণ নতুন, ফাঁকা একটা ক্লাউড মেশিনে বিল্ড চালায় — তাই আগের মতো কোনো
leftover ফাইল মিশে যাওয়ার সুযোগ নেই। GitHub-এর "Actions" ট্যাব থেকে রান
করে "base-apk" আর্টিফ্যাক্ট ডাউনলোড করুন। এই workflow নিজে থেকেই নিচের
চেকগুলো চালিয়ে দেয় — কোনো সমস্যা পেলে build লাল (fail) দেখাবে।

## ম্যানুয়ালি যাচাই করতে চাইলে (Android Studio দিয়ে বিল্ড করলে)
```bash
unzip -o app-release-unsigned.apk -d /tmp/check

# ১) dex ফাইলে com.wevlo.tpl.base ছাড়া অন্য কোনো ক্লাস আছে কিনা —
#    android/java/javax/dalvik/org.json বাদে কিছু থাকলে সেটা সন্দেহজনক
strings /tmp/check/classes*.dex | grep -E '^L[A-Za-z0-9/_$]+;$' \
  | grep -vE '^L(android|java|javax|dalvik|org/json)/' \
  | grep -v '^Lcom/wevlo/tpl/base/'
# ↑ এই কমান্ডের আউটপুট খালি হওয়া উচিত। কিছু দেখালে বিল্ড dirty।

# ২) manifest-এ কোনো অচেনা provider/service/initializer আছে কিনা
strings -e l /tmp/check/AndroidManifest.xml \
  | grep -iE '\.(initializer|provider|worker|analytics|logger)\b'
# ↑ এটারও আউটপুট খালি হওয়া উচিত।

# ৩) শুধু একটা classes.dex থাকা উচিত (ছোট, dependency-free অ্যাপ বলে
#    classes2.dex থাকার দরকার নেই — থাকলে সেটা অস্বাভাবিক, মানে কিছু
#    বাড়তি জিনিস মার্জ হয়ে গেছে)
ls /tmp/check/classes*.dex
```

## তারপর
- সব চেক পাস করলে: এই APK-কে মূল Wevlo প্রজেক্টের
  `app/src/main/assets/template/base.apk`-এ কপি করুন।
- Wevlo দিয়ে একটা টেস্ট অ্যাপ বানান, সাতটা toggle
  (Fullscreen / Exit Confirmation / Pull to Refresh / Pinch Zoom /
  Media Autoplay / PC Mode / Long Press Menu) একে একে অন করে সত্যিকারের
  ডিভাইসে টেস্ট করুন।
- তারপরও কোনোটা crash করলে সেটা এখন গ্যারান্টিড build-contamination না —
  আসল runtime বাগ। সেক্ষেত্রে `adb logcat *:E` থেকে crash-এর ঠিক আগের-পরের
  লগ লাগবে exact কারণ বের করতে।
