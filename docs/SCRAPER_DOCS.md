# Perplexity Browser Scraper - Technical Documentation (v4.8 SAt)

## Overview
This document contains the source code and operational logic for the `perplexity_browser_scraper.py`. It is used by the Central Command ecosystem to perform API-Free research using Playwright and local model analysis.

## Core Logic
The scraper uses two main methods:
1. **CDP (Chrome Debug Protocol)**: Connects to an existing Chrome instance on port 9222 for maximum stealth.
2. **Persistent Context**: Launches a standalone Chrome instance with saved sessions.

## Source Code
```python
import asyncio
import sys
import io
import os
import subprocess
from playwright.async_api import async_playwright

# Force UTF-8 for stdout
if sys.stdout.encoding != 'utf-8':
    sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')

USER_DATA_DIR = os.path.join(os.path.expanduser("~"), ".cc-sessions", "perplexity")

async def get_page_content(page, query):
    """Common logic to fill search box and get answer."""
    search_selector = "#ask-input"
    try:
        # Check if we are on the right page
        if "perplexity.ai" not in page.url:
            await page.goto("https://www.perplexity.ai/spaces/central-command-5dPMu7AZQce2Huzerr225A")
        
        await page.wait_for_selector(search_selector, timeout=30000)
        await asyncio.sleep(2)
        await page.click(search_selector) # Focus the lexical editor
        await asyncio.sleep(1)
        await page.keyboard.type(query, delay=50) # Type via keyboard for better compatibility
        await asyncio.sleep(1)
        await page.keyboard.press("Enter")
        
        print("[*] Waiting for answer...", file=sys.stderr)
        await page.wait_for_selector("div.prose", timeout=60000)
        await asyncio.sleep(5) # Allow streaming
        
        content = await page.inner_text("div.prose")
        return content
    except Exception as e:
        print(f"[!] Error: {e}", file=sys.stderr)
        await page.screenshot(path="scraper_error_debug.png")
        return None

async def scrape_with_cdp(query):
    """Connect to an already running Chrome instance."""
    async with async_playwright() as p:
        try:
            print("[*] Connecting to Chrome via CDP (localhost:9222)...", file=sys.stderr)
            browser = await p.chromium.connect_over_cdp("http://localhost:9222")
            context = browser.contexts[0]
            # Try to find a perplexity page or create one
            page = None
            for p_obj in context.pages:
                if "perplexity.ai" in p_obj.url:
                    page = p_obj
                    break
            if not page:
                page = await context.new_page()
            
            return await get_page_content(page, query)
        except Exception as e:
            print(f"[!] CDP Connection failed. Is Chrome running with --remote-debugging-port=9222?", file=sys.stderr)
            return None

async def scrape_with_persistent(query, setup_mode=False):
    """Run a standalone persistent Chrome context."""
    os.makedirs(USER_DATA_DIR, exist_ok=True)
    async with async_playwright() as p:
        browser_context = await p.chromium.launch_persistent_context(
            USER_DATA_DIR,
            channel="chrome",
            headless=not setup_mode,
            user_agent="Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/122.0.0.0 Safari/537.36",
            viewport={'width': 1280, 'height': 720},
            ignore_default_args=["--enable-automation"],
            args=["--disable-blink-features=AutomationControlled"]
        )
        page = await browser_context.new_page()
        await page.add_init_script("Object.defineProperty(navigator, 'webdriver', {get: () => undefined})")
        
        if setup_mode:
            await page.goto("https://www.perplexity.ai/spaces/central-command-5dPMu7AZQce2Huzerr225A")
            print("[*] Manual Setup: Please log in and press Enter here when done.", file=sys.stderr)
            await asyncio.get_event_loop().run_in_executor(None, sys.stdin.readline)
            await browser_context.close()
            return "Setup complete"
            
        result = await get_page_content(page, query)
        await browser_context.close()
        return result

if __name__ == "__main__":
    query = sys.argv[1] if len(sys.argv) > 1 else ""
    setup = "--setup" in sys.argv
    cdp = "--cdp" in sys.argv
    
    # Clean query from flags
    query = " ".join([a for a in sys.argv[1:] if not a.startswith("--")])

    if setup:
        asyncio.run(scrape_with_persistent("", setup_mode=True))
    elif cdp:
        res = asyncio.run(scrape_with_cdp(query))
        if res: print(res)
        else: sys.exit(1)
    else:
        # Default try CDP first, then persistent
        res = asyncio.run(scrape_with_cdp(query))
        if not res:
            res = asyncio.run(scrape_with_persistent(query))
        
        if res: print(res)
        else: sys.exit(1)
```
