<script>
    import "../app.css";
    import Sun from "@lucide/svelte/icons/sun";
    import Moon from "@lucide/svelte/icons/moon";
    import { Button } from "$lib/components/ui/button/index.js";

    let isDark = false;
    function toggleMode() {
        const htmlEl = document.documentElement;
        const isDark = htmlEl.classList.toggle("dark"); // Toggles "dark" class

        htmlEl.style.colorScheme = isDark ? "dark" : "light";
        localStorage.setItem("theme", isDark ? "dark" : "light");
    }
</script>

<div class="text-3xl font-bold text-foreground">Welcome to SvelteKit</div>

<Button onclick={toggleMode} variant="outline" size="icon" class="relative">
    <Sun class="absolute h-[1.2rem] w-[1.2rem] transition-transform duration-300 dark:rotate-90 dark:scale-0 opacity-100 dark:opacity-0" />
    <Moon class="absolute h-[1.2rem] w-[1.2rem] transition-transform duration-300 rotate-90 scale-0 dark:rotate-0 dark:scale-100 opacity-0 dark:opacity-100" />
    <span class="sr-only">Toggle theme</span>
</Button>


