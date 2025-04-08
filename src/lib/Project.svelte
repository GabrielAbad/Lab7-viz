<script>
    export let data = {};
    export let hLevel = 2;
</script>

<article>
    <svelte:element this={"h" + hLevel}>{data.title}</svelte:element>
    <img src={data.image} alt="" />
    <div>
        <p>
            {data.description}
        </p>
        <p class="year">
            {data.year}
        </p>
    </div>
</article>

<style>
    p {
        font-size: 0.7em;
    }
    img {
        height: 12em;
    }
    p.year{
        font-size: 1.5em;
    }
</style>
