<script>
    import { onMount } from 'svelte';
    import projects from '$lib/projects.json';
    import Project from '$lib/Project.svelte';

    let profileData = null;
    let error = null;

    // Fetch data only on the client
    onMount(async () => {
        try {
            const response = await fetch('https://api.github.com/users/allison-conrey');
            if (!response.ok) throw new Error(`Failed to fetch: ${response.statusText}`);
            profileData = await response.json();
        } catch (err) {
            console.error('Error fetching GitHub data:', err);
            error = err;
        }
    });
</script>

<section class="intro">
    <h1>Welcome to My Portfolio</h1>
    <div class="intro-content">
        <img src="/images/allison.jpg" alt="Allison Conrey, UCSD Data Science graduate student" class="profile-photo">
        <div class="intro-text">
            <h2>Allison Conrey</h2>
            <p>Hello! I'm a graduate student at UCSD specializing in Data Science. Here, you’ll find a showcase of my latest projects and research.</p>
        </div>
    </div>
</section>

<section class="github-stats">
    <h2>My GitHub Stats</h2>
    {#if error}
        <p class="error">Something went wrong: {error.message}</p>
    {:else if profileData}
        <dl class="profile-stats">
            <div class="stat">
                <dt>Followers</dt>
                <dd>{profileData.followers}</dd>
            </div>
            <div class="stat">
                <dt>Following</dt>
                <dd>{profileData.following}</dd>
            </div>
            <div class="stat">
                <dt>Public Repos</dt>
                <dd>{profileData.public_repos}</dd>
            </div>
            <div class="stat">
                <dt>Public Gists</dt>
                <dd>{profileData.public_gists}</dd>
            </div>
        </dl>
    {:else}
        <p>Loading GitHub data...</p>
    {/if}
</section>

<section class="latest-projects">
    <h2>Latest Projects</h2>
    <div class="projects highlights">
        {#each projects.slice(0, 3) as p}
            <Project data={p} hLevel={3} />
        {/each}
    </div>
</section>

<style>
    /* Define color variables for light and dark modes */
    :root {
        --text-color: #333; /* Dark color for light mode */
        --background-color: #f9f9f9; /* Light background for light mode */
    }

    :root[data-theme="dark"] {
        --text-color: #f5f5f5; /* Light color for dark mode */
        --background-color: #222; /* Dark background for dark mode */
    }

    /* Apply background and text color to GitHub stats section */
    .github-stats {
        background-color: var(--background-color);
        color: var(--text-color);
        padding: 2rem 1rem;
        margin-bottom: 2rem;
        text-align: left;
    }

    .github-stats h2 {
        font-size: 1.5rem;
        margin-bottom: 1rem;
        color: var(--text-color);
    }

    .profile-stats {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 2rem;
        text-align: left;
        color: var(--text-color); /* Apply theme-based text color */
    }

    .stat {
        display: flex;
        flex-direction: column;
    }

    .profile-stats dt {
        font-size: 0.9rem;
        color: var(--text-color);
        text-transform: uppercase;
        margin-bottom: 0.2rem;
        font-weight: 500;
        letter-spacing: 1px;
    }

    .profile-stats dd {
        font-size: 2rem;
        font-weight: bold;
        color: var(--text-color);
        margin: 0;
    }

    /* Introductory Section Styling */
    .intro {
        text-align: center;
        padding: 2rem 1rem;
        background-color: #f9f9f9;
        border-radius: 8px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
        margin-bottom: 2.5rem;
        color: #333
    }

    .intro-content {
        display: flex;
        flex-direction: column;
        align-items: center;
        gap: 1rem;
    }

    .profile-photo {
        border-radius: 50%;
        width: 150px;
        height: 150px;
        box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    }

    .intro-text {
        max-width: 600px;
    }

    .intro-text h2 {
        font-size: 1.8rem;
        margin: 0.5rem 0;
        color: #333;
    }

    .intro-text p {
        font-size: 1rem;
        line-height: 1.6;
        color: #666;
    }

    /* GitHub Profile Stats Styling */
    .github-stats {
        padding: 2rem 1rem;
        margin-bottom: 2rem;
        text-align: left; /* Align text to the left */
    }

    .github-stats h2 {
        font-size: 1.5rem;
        margin-bottom: 1rem;
        color: #333;
    }

    .profile-stats {
        display: grid;
        grid-template-columns: repeat(4, 1fr);
        gap: 2rem;
        text-align: left; /* Align the content within each column to the left */
    }

    .stat {
        display: flex;
        flex-direction: column;
    }

    .profile-stats dt {
        font-size: 0.9rem;
        color: #7D7D7D; /* Light grey color */
        text-transform: uppercase;
        margin-bottom: 0.2rem;
        font-weight: 500;
        letter-spacing: 1px;
    }

    .profile-stats dd {
        font-size: 2rem;
        font-weight: bold;
        color: #000;
        margin: 0;
    }

    /* Latest Projects Section Styling */
    .latest-projects {
        text-align: center;
        padding: 2rem 1rem;
    }

    .latest-projects h2 {
        font-size: 2rem;
        margin-bottom: 1.5rem;
        color: #333;
    }

    .projects.highlights {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 1.5rem;
        padding: 1rem;
    }

    .error {
        color: red;
        font-size: 1rem;
    }
</style>



