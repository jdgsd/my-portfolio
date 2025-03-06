<svelte:head>
  <title>Contact Me</title>
</svelte:head>


<h1>Contact</h1>

<form action="mailto:jazzlynnn@yahoo.com" method="get">
    <label for="email">Email </label>
    <input type="email" name="email" id="email" required />

    <label for="subject">Subject</label>
    <input name="subject" id="subject" />

    <label for="body">Body</label>
    <textarea name="body" id="body"></textarea>

    <button>Submit</button>
</form>