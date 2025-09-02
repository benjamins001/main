// scrape-pages.js
// npm install puppeteer

const fs = require("fs");
const puppeteer = require("puppeteer");

async function scrapeCategoryPages(browser, categoryUrl) {
  const page = await browser.newPage();
  let currentUrl = categoryUrl;

  while (true) {
    console.log(`🔎 Visiting: ${currentUrl}`);

    await page.goto(currentUrl, {
      waitUntil: "domcontentloaded",
      timeout: 90000,
    });

    // Save current page immediately
    fs.appendFileSync("pages.txt", currentUrl + "\n", "utf8");
    console.log(`💾 Saved: ${currentUrl}`);

    // Look for NEXT page
    const nextHref = await page.$eval("a[rel='next']", (a) => a.href).catch(() => null);

    if (nextHref) {
      console.log("➡️ Found NEXT page:", nextHref);
      currentUrl = nextHref;
    } else {
      console.log("🚫 No more NEXT page.");
      break;
    }
  }

  await page.close();
}

(async () => {
  const categories = fs
    .readFileSync("categories.txt", "utf8")
    .split("\n")
    .map((l) => l.trim())
    .filter((l) => l.length > 0);

  const browser = await puppeteer.launch({ headless: true });

  for (const url of categories) {
    console.log("\n==============================");
    console.log("📂 Starting category:", url);
    console.log("==============================\n");

    await scrapeCategoryPages(browser, url);
  }

  await browser.close();
  console.log("\n✅ Finished. All pages saved to pages.txt");
})();
