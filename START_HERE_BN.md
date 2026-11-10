# SafeCircle - প্রথমে কী করবেন

সবচেয়ে সহজভাবে চালাতে **Docker Desktop** ব্যবহার করুন।

## 1) Project folder open করুন

VS Code-এ `SafeCircle` folder খুলুন।

## 2) Docker Desktop চালু করুন

তারপর project root terminal-এ:

```bash
docker compose up --build
```

প্রথম build-এ dependency download হওয়ায় কিছু সময় লাগতে পারে।

## 3) Demo data দিন

আরেকটি terminal খুলে:

```bash
docker compose exec backend npm run seed
```

Demo login:

```text
Email: demo@safecircle.local
Password: Demo12345!
```

## 4) Browser

```text
Frontend: http://localhost:5173
Backend:  http://localhost:4000/health
```

## 5) প্রথম test flow

1. Login করুন।
2. Contacts page-এ trusted contact আছে কিনা দেখুন।
3. Dashboard থেকে SOS চাপুন।
4. Browser location permission Allow করুন।
5. 3-second confirmation শেষে SOS activate করুন।
6. Emergency page-এ tracking link দেখাবে/generate করুন।
7. Link copy করে Incognito window-এ খুলুন।
8. Main window থেকে live GPS update এলে Incognito map-এ update দেখার কথা।
9. `I'm safe - resolve session` চাপুন।
10. History page-এ timeline দেখুন।

## গুরুত্বপূর্ণ

Core demo চালাতে paid API key দরকার নেই। Email/SMS notification বর্তমানে development provider-এর মাধ্যমে database-এ record হয় এবং backend console-এ log হয়। AI assistant-ও API key না থাকলে deterministic fallback answer দেয়।

Real email/SMS বা external LLM ব্যবহার করতে provider credentials পরে যোগ করতে হবে।

Browser background location ফোন lock হলে guaranteed নয়। Production mobile app-এর জন্য React Native/native background location দরকার হবে।
