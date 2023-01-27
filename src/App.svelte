<script lang="ts">
  import phrases from "./phrases"

  let showInput = false
	let word = ""
  let started = false
  let ended = false
  let guessedLetters = new Map<string, bool>()
  const letters:string[] = []
  for (let index = 0; index < 26; index++) {
    letters.push(String.fromCharCode(65+index))
  }
  const lettersSet = new Set(letters)
  const guessLetter = (letter:string)=>{
    guessedLetters = new Map(guessedLetters)
    guessedLetters.set(letter, word.split("").findIndex(char=>char==letter)>=0)
  }
  const startGame = () => {
    console.log("game started")
    word = word.toUpperCase()
    started = true
  }
  const newGame = () => {
    showInput = false
  	word = ""
    started = false
    ended = false
    guessedLetters = new Map<string, bool>()
  }
  const startWithRandom = () => {
    word = phrases[Math.floor(Math.random()*phrases.length)]
    startGame()
  }
  const inputkeyUpHandler = (event:KeyboardEvent) => {
    if(event.key == "Enter") startGame()
  }
  const mainKeyUpHandler = (event:KeyboardEvent) => {
    console.log(event)
    if(ended || !started) return
    const key = event.key.toUpperCase()
    if(!lettersSet.has(key)) return
    console.log(key)
    guessLetter(key)
  }
  document.addEventListener("keydown", mainKeyUpHandler)
  $: wordLetters = new Set(word.replaceAll(" ","").split("").filter(letter=>lettersSet.has(letter)))
  $: correctLetters = new Set(Array.from(guessedLetters).filter(e=>e[1]).map(e=>e[0]))
  $: incorrectLetters = new Set(Array.from(guessedLetters).filter(e=>!e[1]).map(e=>e[0]))
  $: ended = correctLetters.size == wordLetters.size || incorrectLetters.size >= 8
  $: toShow = word.split("").map(
      letter => (!lettersSet.has(letter) || guessedLetters.has(letter)) ? letter : "_"
  ).join("")
</script>

<main>
  {#if !started}
    <div class="phraseInput">
      <h1>Hangman</h1>
      <div class="inputsDiv">
        {#if showInput}
          <input type="text" bind:value={word} placeholder="Write a word or phrase" on:keyup={inputkeyUpHandler} />
        {:else}
          <input type="password" bind:value={word} placeholder="Write a word or phrase" on:keyup={inputkeyUpHandler} />
        {/if}
        <input type="button" class="btn" value="start" on:click={startGame} disabled={word.length == 0} />
      </div>
        <label>
          <input type="checkbox" bind:checked={showInput}>
          Show phrase
        </label>
      <span>or</span>
      <input type="button" class="btn" value="Random phrase" on:click={startWithRandom}>
    </div>
  {:else}
  <div class="horizontal">
    <h1 class="word">
      {ended ? word : toShow}
    </h1>
    <div class="hangman">
      <div class="hangmanImage">
        {incorrectLetters.size}/8 wrong answers
      </div>
      {#if correctLetters.size == wordLetters.size}
        <h1 class="wintext">You Win!</h1>
      {:else if incorrectLetters.size >= 8}
        <h1 class="losetext">You Lose!</h1>
      {/if}
      {#if ended}
        <input type="button" class="btn" value="New Game" on:click={newGame} />
      {/if}
    </div>
  </div>
  <div class="letters">
    {#each letters as letter}
      <input type="button"
        class="letter" 
        value={letter} 
        disabled={ended || guessedLetters.has(letter)}         
        on:click={()=>guessLetter(letter)}
        data-correct={guessedLetters.get(letter)}
      />
    {/each}
  </div>
  {/if}
</main>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@100;200;300;400;500;600;700;800;900&display=swap');
  :root{
    font-family: sans-serif;
    overflow-x: hidden;
    font-family: 'Inter', sans-serif;
  }
  main{
    display: flex;
    flex-direction: column;
    justify-content: center;
    gap: 10px;
    min-height: 80vh;
  }
  .word{
    letter-spacing: 5px;
    font-family: monospace;
    text-align: center;
    font-size: 3.4em;
    padding: 1em 2em;
  }
  .hangman{
    aspect-ratio: 1 / 1;
    display: flex;
    flex-direction: column;
    justify-content: center;
  }
  .letters{
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    justify-content: center;
    max-width: 800px;
    margin: 0 auto;
  }
  .letter{
    aspect-ratio: 1 / 1;
    box-sizing: border-box;
    width: 40px;
    border: 0;
    border-radius: .5em;
  }
  .letter[data-correct="true"]{
    background-color: #5f5;
  }
  .letter[data-correct="false"]{
    background-color: #f55;
  }
  .horizontal{
    display: flex;
    width: 100%;
    justify-content: space-evenly;
    align-items: center;
    flex-wrap: wrap-reverse;
  }
  .wintext{
    text-align: center;
  }
  .losetext{
    text-align: center;
  }
  .phraseInput{
    width: 100%;
    height: 80vh;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    margin: 20px 0;
    gap: 10px;
  }
  .phraseInput > h1{
    font-size: 2.3em;
    font-weight: 900;
  }
  .inputsDiv{
    display: flex;
    border-radius: .2em;
    overflow: hidden;
    box-shadow: 0 2px 20px #0003;
    max-width: 80vw;
  }
  .btn, .phraseInput input{
    box-sizing: border-box;
    border: 0;
    padding: .5em;
    font-size: 1.4em;
    outline: 0;
    box-sizing: border-box;
    border-radius: .2em;
  }
  .phraseInput input[type="text"], .phraseInput input[type="password"]{
    width: 100%;
  }
  .inputsDiv input[type="button"]:not(:disabled):hover{
    background-color: #38f;
    color: #fff;
  }
  .phraseInput > input{
    border-radius: .2em;
    background-color: #f40;
    color: #fff;
    font-weight: 700;
  }
  .phraseInput input[type="button"]{
    cursor: pointer;
  }
  
</style>
