<script>
    import { page } from '$app/stores';

    const navLinks = [
        { title: 'Home', href: '/' },
        { title: 'Projects', href: '/projects' },
        { title: 'Contact', href: '/contact' },
        { title: 'Meta', href: '/meta'},
        { title: 'Resume', href: 'https://docs.google.com/document/d/1MvO_SPaArLqPTd3fcjVUf6nAVo75bHhgX67YEaoC2dY/view', external: true },
        { title: 'GitHub', href: 'https://github.com/allison-conrey', external: true }
    ];

    // Check for existing color scheme in localStorage or set default
    let localStorage = globalThis.localStorage ?? {}; // Handle localStorage in server environment
    let colorScheme = localStorage.colorScheme ?? 'light dark';

    // Reference the <html> element
    let root = globalThis?.document?.documentElement;

    // Update color scheme on <html> element and save preference
    $: root?.style.setProperty('color-scheme', colorScheme);
    $: localStorage.colorScheme = colorScheme;
</script>

<nav>
    {#each navLinks as link}
        <a
            href={link.href}
            target={link.external ? "_blank" : "_self"}
            class:current={$page.url.pathname === link.href}
        >
            {link.title}
        </a>
    {/each}
</nav>

<label class="color-scheme">
    Theme:
    <select bind:value={colorScheme}>
        <option value="light dark">Automatic</option>
        <option value="light">Light</option>
        <option value="dark">Dark</option>
    </select>
</label>

<slot />

<style>
    nav {
        display: flex;
        justify-content: space-around;
        padding: 1rem;
        background-color: #f5f5f5;
        border-bottom: 1px solid #ddd;
    }
    nav a {
        text-decoration: none;
        color: #333;
        padding: 0.5rem;
    }
    nav a.current {
        border-bottom: 2px solid #333;
        color: #000;
        font-weight: bold;
    }
    nav a:hover {
        color: #555;
    }
    .color-scheme {
        position: absolute;
        top: 1rem;
        right: 1rem;
        font-size: 0.8rem;
        font-family: inherit;
    }
    .color-scheme select {
        padding: 0.3em;
        font: inherit;
    }
</style>
