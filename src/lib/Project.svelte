<script>
    export let data = {};
    export let hLevel = 2;
</script>

<article>
    <svelte:element this={"h" + hLevel}>{data.title}</svelte:element>
    <img src={data.image} alt="">
    <div>
        <p>
            {data.description}
        </p>
        <div class="projyr">
            <p style="color: oklch(80% 3% 200)">
                c. {data.year}
             </p>
        </div>
    </div>
</article>