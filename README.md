# How-to-bypass-Amazon-Waf-Captcha-Even-with-Almost-Zero-Development-Experience
<p>A brief epigraph — if you’re creating an instruction, make sure it covers everything, as in the instruction on how to solve the Amazon captcha (https://2captcha.com/p/amazon-captcha-bypass); otherwise, a junior might break a leg</p>
<p>So, what’s all this about? Well, when I needed to solve the Amazon captcha, the notorious Waf Captcha, I went to dig into a service that I constantly use when working with SEO Autopilot Software and some other services (2captcha) (https://2captcha.com/?from=2957847).</p>
<p>And I found an instruction there, the link to which I provided above. As you probably understand from the epigraph, I didn’t understand a thing, or rather, I understood that I need to use an API, but that’s about it…</p>
<p>It was actually easier with Selenium (https://www.reddit.com/r/Python/comments/18dfcbp/how_to_bypass_recaptcha_in_selenium_automatically/).</p>
<p>The main problem is the short timeout allocated for solving the captcha on Amazon’s side. The time to solve the captcha is limited, and if there is no response, the captcha refreshes (it updates 2 parameters — <code>iv and context</code>).</p>
<p>This means that the freshness timeout of the captcha is about 30 seconds, and within this time, you need to find the parameters on the page, copy them, paste them into the script code, and run it. After this, 2captcha should solve it and return the correct answer. I tried to do this for a couple of unsuccessful hours, developed a sequence of actions, but unfortunately, searching and replacing the changing parameters takes at least 12–15 seconds, leaving 15 to 18 seconds for the captcha solving service, which in today’s reality sounds quite fantastical.</p>
<p>A different approach is needed here; the script should search and substitute parameters. But how to write it for someone who has never seen anything more complex than Ahrefs in their life?</p>
<p>For this very reason, I believe that instructions like those mentioned in the article need to be more detailed than “Just use the API, are you completely crazy?”</p>
<p>Solution In the end, a solution was found, and it took me about 3 hours. Let me explain how a junior can solve the Amazon captcha automatically.</p>
<p>We’ll need GPT Chat and, in my case, video recognition of the Amazon captcha (you won’t need it, as I’ll provide you with the already assembled file).</p>
<p>I got the video from a friend programmer, but he forbade me from posting it publicly because it’s not anonymized.</p>
<p>But you won’t leave me alone, you’ll ask for proofs, and then I’ll have to resort to antidepressants because no one believes me. Therefore, I reproduced this video at the very end when I got the finished script, and I’ll gladly attach it at the end of this text as a demonstration of my endless dedication to the audience!</p>
<p>So, let’s go in order:</p>
<p>I took the video, made 3 screenshots, and uploaded them to GPT Chat, asking it to rewrite this code for me in text.</p>
<p>There were several files in the video, but only the contents of 2 of them were shown — index.js and inject.js — that’s where I started.</p>
<p>I won’t bore you with a transcript of the screenshots (I had to do a bit of work to assemble the code from the screenshots, but in the end, I got two pieces of code for two files like this:</p>
<per>// index.js
import { launch } from 'puppeteer'
import { Captcha } from '2captcha-ts'
import { readFileSync } from 'fs'

const solver = new Captcha(process.env.APIKEY)

const target = 'URL where CAPTCHA occurs'

const example = async () => {
  const browser = await launch({
    headless: false,
    devtools: true
  })

  const [page] = await browser.pages()

  const preloadFile = readFileSync('./inject.js', 'utf8')
  await page.evaluateOnNewDocument(preloadFile)

  // Here we intercept the console messages to catch the message logged by inject.js script
  page.on('console', async (msg) => {
    const txt = msg.text()
    if (txt.includes('intercepted-params:')) {
      const params = JSON.parse(txt.replace('intercepted-params:', ''))

      const wafParams = {
        pageurl: target,
        sitekey: params.key,
        iv: params.iv,
        context: params.context,
        challenge_script: params.challenge_script,
        captcha_script: params.captcha_script
      }
      console.log(wafParams)

      try {
        console.log('Solving the captcha...')
        const res = await solver.solveRecaptchaV2(wafParams)
        console.log(`Solved the captcha ${res.id}`)
        console.log(res)
        console.log('Using the token...')
        await page.evaluate(token => {
          window.localStorage.inputCaptchaToken(token)
        }, res.data.captcha.voucher)
        console.log(e)
      } catch (e) {
        console.log(e)
      }
    }
  })

  await page.goto(target)
  // Additional code to interact with the page after captcha is solved might be here...
}

example()
</per>
<p>And the second file, <code>inject.js</code></p>
<per>
  console.clear = () => console.log('Console was cleared')

const i = setInterval(() => {
  if (window.CaptchaScript) {
    clearInterval(i)

    let params = gokProps

    Array.from(document.querySelectorAll('script')).forEach(s => {
      const src = s.getAttribute('src')
      if (src && src.includes('captcha.js')) params.captcha_script = src
      if (src && src.includes('challenge.js')) params.challenge_script = src
    })

    console.log('intercepted-params: ' + JSON.stringify(params))
  }
}, 5)
</per>
<p>Naturally, I clarified with the Chat how to make the code work and received a recommendation to use the standard command:</p>
<per>
  node index.js
</per>
<p>But I’ve been in these internet spaces for a very long time, and I understand that the code won’t just work if the necessary libraries aren’t installed on the computer. My neuroconsultant helped me here as well.</p>
<p>After studying the code I provided in the form of screenshots, he recommended installing the following packages:</p>
<p>puppeteer (https://www.npmjs.com/package/puppeteer)</p>
<p>2captcha-ts (https://www.npmjs.com/package/2captcha-ts)</p>
<p>And accordingly, the installation code:</p>
<per>
  npm install puppeteer 2captcha-ts
</per>
<p>However, I couldn’t install them all together via VS Code, probably due to my clumsy hands. I installed them one by one through the console:</p>
<per>
  npm install puppeteer
npm install 2captcha-ts
</per>
<p>The next steps are more interesting because everything before this stage I somehow know, but the next block is a real mystery to me, and I just do what my neuroconsultant tells me.</p>
<p>So, for the code to work correctly, a file with the .env encoding was needed, and it could be without a name. This file should contain the following data:</p>
<per>
  APIKEY=your_2captcha_api_key
</per>
<p>Clearly, instead of the parameter <code>your_2captcha_api_key</code>, I inserted my key from the service.</p>
<p>There were six more recommendations on what needs to be done for the code to work, but</p>
<p>And the first run and the first error.</p>
<p>The error was related to using ES6 import syntax in Node.js. To use it, you need to either specify the module type in package.json or change the file extension to <code>.mjs</code>.</p>
<p>I didn’t feel like specifying the module type in package.json, so I took the path of least resistance and simply renamed the file extension from js to mjs.</p>
<p>The next error was related to the imported package 2captcha-ts, but I don’t want to describe them in detail. I’ll just say that I changed the code responsible for launching this module several times, added logging, and checked the correctness of connecting the .env file.</p>
<p>And it still didn’t work; the script continued to throw an error and stubbornly refused to run. I compared the screenshot and the code that Chat pulled from there and found a small typo, which was the root of the problem.</p>
<p>But, as it turned out, even this didn’t help — the error <code>"APIKEY is not defined in the .env file"</code> occurred.</p>
<p>We fixed this by adding a new package:</p>
<p>dotenv(https://www.npmjs.com/package/dotenv)</p>
<per>
  npm install dotenv
</per>
<p>After that, a couple more typos surfaced, which Chat tried to solve by changing the code structure, but it was just necessary to carefully compare the screenshot and the code itself.</p>
<p>And finally, the code for the index.js file ended up like this:</p>
<per>
  // index.js
import { launch } from 'puppeteer'
import { Captcha } from '2captcha-ts'
import { readFileSync } from 'fs'

const solver = new Captcha(process.env.APIKEY)

const target = 'URL where CAPTCHA occurs'

const example = async () => {
  const browser = await launch({
    headless: false,
    devtools: true
  })

  const [page] = await browser.pages()

  const preloadFile = readFileSync('./inject.js', 'utf8')
  await page.evaluateOnNewDocument(preloadFile)

  // Here we intercept the console messages to catch the message logged by inject.js script
  page.on('console', async (msg) => {
    const txt = msg.text()
    if (txt.includes('intercepted-params:')) {
      const params = JSON.parse(txt.replace('intercepted-params:', ''))

      const wafParams = {
        pageurl: target,
        sitekey: params.key,
        iv: params.iv,
        context: params.context,
        challenge_script: params.challenge_script,
        captcha_script: params.captcha_script
      }
      console.log(wafParams)

      try {
        console.log('Solving the captcha...')
        const res = await solver.solveRecaptchaV2(wafParams)
        console.log(`Solved the captcha ${res.id}`)
        console.log(res)
        console.log('Using the token...')
        await page.evaluate(token => {
          window.localStorage.inputCaptchaToken(token)
        }, res.data.captcha.voucher)
        console.log(e)
      } catch (e) {
        console.log(e)
      }
    }
  })

  await page.goto(target)
  // Additional code to interact with the page after captcha is solved might be here...
}

example()
</per>
<p>transformed into this index.mjs:</p>
<per>
  // index.js
import 'dotenv/config';
import { launch } from 'puppeteer'
import Captcha from '2captcha-ts';
import { readFileSync } from 'fs'

// Checking for APIKEY in .env
if (!process.env.APIKEY) {
    console.error("APIKEY is not defined in .env file");
    process.exit(1); // Terminate execution if key not found
}

const solver = new Captcha.Solver(process.env.APIKEY);

const target = 'URL where CAPTCHA occurs'

const example = async () => {
    const browser = await launch({
        headless: false,
        devtools: true
    })

    const [page] = await browser.pages()

    const preloadFile = readFileSync('./inject.js', 'utf8')
    await page.evaluateOnNewDocument(preloadFile)

    // Here we intercept the console messages to catch the message logged by inject.js script
    page.on('console', async (msg) => {
        const txt = msg.text()
        if (txt.includes('intercepted-params:')) {
            const params = JSON.parse(txt.replace('intercepted-params:', ''))

            const wafParams = {
                pageurl: target,
                sitekey: params.key,
                iv: params.iv,
                context: params.context,
                challenge_script: params.challenge_script,
                captcha_script: params.captcha_script
            }
            console.log(wafParams)

            try {
                console.log('Solving the captcha...')
                const res = await solver.amazonWaf(wafParams)
                console.log(`Solved the captcha ${res.id}`)
                console.log(res)
                console.log('Using the token...')
                await page.evaluate(async (token) => {
                    await ChallengeScript.submitCaptcha(token);
                    window.location.reload ()
                }, res.data.captcha_voucher);
            } catch (e) {
                console.log(e)
            }
        } else {
            return
        }
    })

    await page.goto(target)
    // Additional code to interact with the page after captcha is solved might be here...
}

example()
</per>
<p>In the code above, don’t forget to substitute the correct URL where captcha solving is required (if you decide to apply it).</p>
<p>And from this inject.js:</p>
<per>
  console.clear = () => console.log('Console was cleared')

const i = setInterval(() => {
  if (window.CaptchaScript) {
    clearInterval(i)

    let params = gokProps

    Array.from(document.querySelectorAll('script')).forEach(s => {
      const src = s.getAttribute('src')
      if (src && src.includes('captcha.js')) params.captcha_script = src
      if (src && src.includes('challenge.js')) params.challenge_script = src
    })

    console.log('intercepted-params: ' + JSON.stringify(params))
  }
}, 5)
</per>
<p>To this inject.js:</p>
<per>
  // console.clear = () => console.log('Console was cleared')

let counter = 0;
const MAX_ATTEMPTS = 2000; // Approximately 10 seconds at 5 ms intervals
console.log('Let's start searching for CaptchaScript...');

const i = setInterval(() => {
    console.log(`Попытка ${counter}: Checking for CaptchaScript...`);

    if (window.CaptchaScript || counter > MAX_ATTEMPTS) {
        clearInterval(i);
        if (!window.CaptchaScript) {
            console.log('CaptchaScript не найден');
        } else {
            console.log('CaptchaScript найден');

            let params = gokuProps;
            Array.from(document.querySelectorAll('script')).forEach(s => {
                const src = s.getAttribute('src');
                if (src && src.includes('captcha.js')) params.captcha_script = src;
                if (src && src.includes('challenge.js')) params.challenge_script = src;
            });

            console.log('intercepted-params: ' + JSON.stringify(params));
        }
    }
    counter++;
}, 5);
</per>
<p>And the script worked. Like, completely. It solved the captcha and continues to solve it with each launch.</p>
(https://youtu.be/D2hHcy3Hizw)
<p>So, it seems like the future has arrived, old man. But not quite.</p>
<p>What should you do with this information? Well, darn, I really like the advice — live with it now. But I don’t want to give advice. Look for yourself, maybe someone will find it useful for their projects…</p>






