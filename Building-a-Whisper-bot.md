# بناء بوت قراءة الملفات الصوتية بإستخدام Whisper

في هذا الدرس، سننشئ بوتًا بسيطًا يحصل على نسخ من الملفات الصوتية ويقوم بترجمتها الى نصوص. سنستخدم اداة Whisper المُقدمة من OpenAI لهذا الغرض.

## التقنيات

سنستخدم في هذا الدرس إطار العمل Nextjs ولغة TypeScript وإطار Tailwind.

تأكد من تثبيت Nodejs بالفعل على جهازك. إذا لم تقم بتنزيله يمكنك تنزيله <a href="https://nodejs.org/en/download" target="_blank">من هنا</a>

سنستخدم أحدث إصدار من Nextjs، والذي يتضمن استخدام موجه التطبيقات. إنه مختلف تمامًا عن الإصدار الذي قد تكون معتادًا عليه حاليًا.

تحتاج أيضًا إلى مفتاح OpenAI API. إذا لم يكن لديك، فإحصل على واحدة <a href="https://platform.openai.com/account/api-keys" target="_blank">من هنا</a>

> في حال تريد الوصول الى المشروع بشكل مباشر يمكنك الذهاب من هنا
> <a href="https://github.com/Web3Arabs/whisperbot.git" target="_blank">https://github.com/Web3Arabs/whisperbot.git</a>

قم بإستدعاء مشروع Nextjs من خلال إضافة هذا في terminal:

```bash
npx create-next-app@latest
```

والاستمرار في تتبع الخطوات وتثبيت اي package قد يطلبها منك.

اسم المشروع: **whisperbot**

تأكد من أن إعدادات مشروعك تبدو بهذا الشكل:
<img src="https://www.web3arabs.com/courses/ai/whisperbot/whisperbot-next.png"/>

قم بتثبيت حزمة dotenv حتى تتمكن من إستيراد ملف **env.**

```bash
npm install dotenv
```

الان قم بإنشاء ملف **env.** في المشروع الذي قمنا بإنشائه واضف **secret key** الخاص بك

```bash
OPENAI_API_KEY="sk-yoursecretkey"
```

> يمكنك الحصول على secret key الخاص بك <a href="https://platform.openai.com/account/api-keys" target="_blank">من هنا</a>

## الواجهة الأمامية (Front-end)

بعد إن قمنا بإستدعاء مشروع Nextjs وإعداد بيئة العمل الذي سنعمل عليها دعونا نبدء في البناء

إذهب إلى الملف **src/app/globals.css** وقم بإضافة هذا الكود بعد ان تقوم بإزالة الكود السابق

```CSS
@tailwind base;
@tailwind components;
@tailwind utilities;
```

إذهب إلى ملف **src/app/page.tsx** وقم بإزالة كل الكود الذي ستراه واضف هذا الكود وقم بإتباع كل سطر كما هو موضح وظيفة كل شيء متواجد في الكود.

```TSX
"use client";
// في الصفحة فيجب ان نجعلها تُعرض من جانب العميل وليس السيرفر useState بما اننا سنقوم بإضافة
import React, { useState } from "react";

export default function Home() {
  // للحصول على الملف الذي يدخله المستخدم
  const [theFile, setTheFile] = useState<File | null>(null);
  // OpenAI ستظهر للمستخدم نص اثناء الإنتظار لرد
  const [isLoading, setIsLoading] = useState(false);
  // وسنظهره في الصفحة OpenAI لتخزين الرد الذي سنحصل عليه من
  const [response, setResponse] = useState("");

  // التفاعل مع التعديلات التي تحدث في خانة رفع الملفات
  const handleFileChange = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.currentTarget.files?.[0];
    if (!file) return;

    // لأي شيء يقوم المستخد بتحميله theFile تعيين حالة
    setTheFile(file);
  };

  /**
   * OpenAI تقوم هذه الدالة بالإتصال برد
   */
  const getTranscription = async () => {
    // يظهر للمستخدم انه ما زال البوت يقوم بالرد وعليه الإنتظار حتى ينتهي
    setIsLoading(true);

    // إذا لم يكن هناك ملف فإن الدالة لن تعيد أي شيء
    if (!theFile) {
      setIsLoading(false);
      return;
    }
    const formData = new FormData();
    formData.set("file", theFile);

    try {
      // الخاص بنا API نقوم بإرسال طلب الى
      const response = await fetch("/api", {
        method: "POST",
        body: formData,
      });

      // التعامل مع حالة الإرسال
      if (response.ok) {
        console.log("File uploaded!");
      } else {
        console.error("Failed!");
      }

      const data = await response.json();
      // الذي سنعرضه في الصفحة response يقوم بتخزين المخرجات الى المتغير
      setResponse(data.output.text);
    } catch (error) {
      console.error(error);
    }

    setTheFile(null);
    setIsLoading(false);
  };

  return (
    <main dir="rtl" className="m-2">
      <div>
        <div>
          <input type="file" accept=".wav, .mp3" onChange={handleFileChange} />
          <div>
            {isLoading ? "يرجى الإنتظار..." : response ? response : ""}
          </div>
        </div>
        <div>
          <button
            onClick={getTranscription}
            className="bg-red-900 text-white px-8 py-3 mt-2 rounded-sm"
          >
            رفع الملف
          </button>
        </div>
      </div>
    </main>
  );
}
```

قم بتشغيل المشروع لترى ما الذي سيظهر امامك

```bash
npm run dev
```

هذا ما سيظهر لك اثناء الذهاب الى <a href="http://localhost:3000" target="_blank">localhost:3000</a>

<img src="https://www.web3arabs.com/courses/ai/whisperbot/front-end.png"/>

ستجد أنه ما زال لا يعمل!! والسبب أنه يتبقى لنا العمل على الواجهة الخلفية ويُعد أهم جزء لدينا.

## الواجهة الخلفية (Back-end)

إذهب الى مجلد **src/app** وقم بإنشاء مجلد بإسم **api** وفي هذا المجلد قم بإنشاء ملف بإسم **route.ts**

قم بنسخ هذا الكود وإضافته هناك وقم بإتباع كل سطر كما هو موضح اعلى كل سطر.

```TS
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest, res: NextResponse) {
    // req.formData() متغير يقوم اولاً بتخزين
    const data = await req.formData();
    const theFile: File | null = data.get("file") as unknown as File;

    const formData = new FormData();
    formData.append("file", theFile);
    formData.append("model", "whisper-1");

    /**
     * الذي يقوم بالتعامل مع الملفات التي سيقوم المستخدم برفعها api التواصل مع
     * OPENAI_API_KEY تهيئة المشروع بمفتاح
     */
    const response = await fetch(
        "https://api.openai.com/v1/audio/transcriptions",
        {
            method: "POST",
            headers: {
                Authorization: `Bearer ${process.env.OPENAI_API_KEY}`,
            },
            body: formData,
        }
    );

    const body = await response.json();
    // نقوم بإعادة المخرجات إلى الواجهة الامامية
    return NextResponse.json({ output: body });
}
```

قم بتجربته الان على <a href="http://localhost:3000" target="_blank">localhost:3000</a>

قم برفع اي تسجيل صوتي لديك وانقر على الزر "رفع الملف" وانتظر النتيجة..

عمل رائع! إنه يعمل بشكل جيد 🥳🥳

كما هو الحال دائمًا، إذا كانت لديك أي أسئلة أو شعرت بالتعثر أو أردت فقط أن تقول مرحبًا، فقم بالإنضمام على <a href="https://t.me/Web3ArabsDAO" target="_blank">Telegram</a> او <a href="https://discord.gg/ykgUvqMc4Q" target="_blank">Discord</a> وسنكون أكثر من سعداء لمساعدتك!
