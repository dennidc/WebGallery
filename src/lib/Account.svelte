<script lang="ts">
    import { onMount } from "svelte";
    import type { AuthSession } from "@supabase/supabase-js";
    import { supabase } from "../supabaseClient";

    export let session: AuthSession;

    let loading = false;
    let korean = false;
    let images: { url: string, name: string }[] = [];

    // Function to fetch images from Supabase storage
    const getImages = async () => {
        try {
            loading = true; // Set loading to true at the start of the fetch process
            const { user } = session; // Extract user from the session object

            // Fetch list of images from the storage bucket
            const { data: files, error: listError } = await supabase.storage
                .from("images")
                .list(user.id + "/");

            if (listError) throw listError; // If there's an error, throw it

            // Generate public URLs for each file and store them with their names
            const urls = await Promise.all(
                files.map(async (file) => {
                    const filePath = `${user.id}/${file.name}`;
                    const { data: signedUrlData, error: urlError } =
                        await supabase.storage
                            .from("images")
                            .createSignedUrl(filePath, 60);

                    if (urlError) {
                        throw urlError; // If there's an error, throw it
                    }

                    return { url: signedUrlData.signedUrl, name: file.name };
                }),
            );
            
            images = urls; // Assign the generated URLs and names to the images array
        } catch (error) {
            if (error instanceof Error) {
                alert(error.message); // Alert the error message if an error occurs
            }
        } finally {
            loading = false; // Set loading to false after the fetch process completes
        }
    };

    function setkorean() {
        korean = !korean;
    }

    // Fetch images when the component is mounted
    onMount(() => {
        getImages();
    });

    // Function to handle logout
    const handleLogout = async () => {
        const { error } = await supabase.auth.signOut();
        if (error) {
            alert(error.message); // Alert the error message if an error occurs
        } else {
            // You can redirect the user to a login page or a different route
            window.location.href = '/';
        }
    };
</script>

{#if !korean}
<h1 class="titulo" >Galeria de imágenes</h1>
{:else}
<h2>イメージギャラリー</h2>
{/if}
<br><br>
<!-- Display loading message or images based on the loading state -->
<button on:click={setkorean} class="reload-button">Change to Japanese</button>
<br><br>
{#if loading}
    <p>Loading...</p>
{:else}
    <div class="image-gallery">
        {#each images as { url, name }}
            <div class="image-item">
                <img src={url} alt="User image" />
                <p>{name}</p>
            </div>
        {/each}
    </div>
{/if}

<!-- Logout button -->
<button on:click={handleLogout} class="logout-button">Logout</button>
<button on:click={getImages} class="reload-button">Reload</button> 

<style>
    .image-gallery {
        display: flex;
        flex-wrap: wrap;
        gap: 10px;
        justify-content: space-between;
        text-align: center;
    }
    .image-item {
        display: flex;
        flex-direction: column;
        align-items: center;
    }
    .image-gallery img {
        max-width: 200px;
        height: auto;
        padding: 10px;
        border: 2px white solid;
        
    }
    .image-item p {
        margin-top: 5px;
        font-size: 14px;
        color: white;
    }
    .logout-button {
        margin-top: 20px;
        padding: 10px 20px;
        background-color: #f44336;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
    }
    .reload-button{
        margin-top: 20px;
        padding: 10px 20px;
        border-radius: 5px;
        cursor: pointer;
    }
    .logout-button:hover {
        background-color: #d32f2f;
    }
    .reload-button:hover {
        background-color: white;
        color: black;
    }
</style>
