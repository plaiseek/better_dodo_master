After reading the code, it appear that human verification is only performed by the server based on activity periods, nothing client side.
The following script verifies the webpage JavaScript has not changed before starting.
Use it wisely and don't be greedy.

Read the 'Terms of Use' before not using this script, at your own risks.

 - Open the browser's console (Ctrl+Shift+K on Firefox)
 - Paste the code below
 - Tune `short_wait_range`, `long_wait_range` and `long_wait_prob` for a target efficiency (exceeding 70% is not recommended)
 - Click Run
 - Sleep
 - Reload or close the page to stop

```javascript
function sleep_ms(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
function pick_ms(min_ms, max_ms, n) {
    // Bates distribution, uniform for n=1, bell shaped when n -> +inf
    let r = 0;
    for (let i = 0; i < n; i++) r += Math.random();
    return (min_ms + (r / n) * (max_ms - min_ms));
}
function ihm_wait(min_ms, max_ms) {
    return sleep_ms(pick_ms(min_ms, max_ms, 4));
}
function afk_wait(wait_range_mn) {
    const ms =
        pick_ms(wait_range_mn[0] * 60_000, wait_range_mn[1] * 60_000, 3);
    // Print the next wake time
    const wake = new Date(Date.now() + ms);
    console.log(`next wake: ${wake.toLocaleTimeString('fr-FR')}`);
    return sleep_ms(ms);
}

async function open_pack() {
    // Click on pack
    document.querySelector('main button:has(> img)').click();
    // Wait and get pop-up buttons
    await ihm_wait(2000, 2500);
    let tries = 0;
    let buttons = document.querySelectorAll('main button');
    while (buttons.length != 9) {
        if (tries++ == 20)
            throw new Error('Unexpected buttons count and max tries reached.');
        await sleep_ms(500);
        buttons = document.querySelectorAll('main button');
    }
    await sleep_ms(500);
    const next_btn = buttons[buttons.length - 2];
    const continue_btn = buttons[buttons.length - 1];
    // Click next 4 times
    for (let i = 0; i < 4; i++) {
        next_btn.click();
        await ihm_wait(500, 1000);
    }
    // Wait longer for the last (rarest) card
    await ihm_wait(1200, 1800);
    // Click continue
    continue_btn.click();
}
function read_packs_count() {
    const count_span = document.querySelectorAll('main span')[1];
    const packs_count = parseInt(count_span.textContent, 10);
    if (Number.isNaN(packs_count))
        throw new Error('Failed to read packs count.');
    return packs_count;
}
async function opening_loop() {
    while (true) {
        while (read_packs_count() > 0) {
            await open_pack();
            await ihm_wait(1200, 1500);
        }
        await afk_wait((Math.random() < long_wait_prob)
            ? long_wait_range
            : short_wait_range);
    }
}

// Waiting configuration to appear natural
const short_wait_range = [10, 28]; // in minutes (should be lossless)
const long_wait_range = [55, 80]; // in minutes (should be lossy)
const long_wait_prob = 0.25;     // probability of long wait
// Efficiency calculation (cosmetic)
const is_pro = document.body.innerHTML.includes("Pack PRO du jour");
const wait_between_packs = is_pro ? 3 : 10; // pro = 3 mins, free = 10 mins
const avg_short_wait = (short_wait_range[0] + short_wait_range[1]) / 2;
const avg_long_wait = (long_wait_range[0] + long_wait_range[1]) / 2;
const avg_long_wait_lost = (avg_long_wait / wait_between_packs) - 10;
const avg_wait = (1 - long_wait_prob) * avg_short_wait
    + long_wait_prob * avg_long_wait;
const avg_lost_per_wait = long_wait_prob * avg_long_wait_lost;
const avg_packs_per_wait = (avg_wait / wait_between_packs) - avg_lost_per_wait;
const efficiency = 100 * (avg_packs_per_wait / avg_wait) * wait_between_packs;

if ([...document.querySelectorAll("script[src]")]
    .filter(s => s.src.includes("0tukeil_.xtnj.js")).length == 1) {
    console.log(`Started with target efficiency of ${efficiency.toFixed(2)}%.`);
    console.log("Reload page to interrupt.");
    opening_loop().catch(err => console.error('aborted:', err.message));
} else {
    console.log("The website scripts changed, unsafe to bot. (try to reload?)");
}
```
