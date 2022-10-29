<script>
  // FOR CONVERTING WORDS TO NUMBERS

  import ltn from '$lib/letters-to-numbers.json'

  function convertToNumber(words) {
    let pn = ''
    if (words !== undefined && words.length !== 0) {
      words.split('').forEach(letter => {
        const num = ltn[letter]
        pn += num == undefined ? ' ' : num
      })
      return pn
    } else {
      return '---------'
    }
  }

  let words
  $: foundNumber = convertToNumber(words)

  // FOR CONVERTING NUMBERS TO WORDS
  let number = ''

  let perfect

  import ntw from '$lib/numbers-to-words.json'

  function generateBeginningPossibilities(text) {
    let variations = []
    let end = 0
    while (end < text.length + 1) {
      variations.push(text.slice(0, end))
      end++
    }

    return variations
  }

  function generatePerfectOutcomes(outcomes, completed) {
    let found = []
    outcomes.forEach(outcome => {
      let words = []

      if (outcome.remaining.length == 0) {
        completed.push(outcome)
      }

      generateBeginningPossibilities(outcome.remaining).forEach(possibility => {
        let word = ntw[possibility]
        if (word != undefined) {
          word.forEach(w => {
            words.push({
              word: w,
              subject: possibility
            })
          })
        }
      })
      words.forEach(wrd => {
        let toPush = [
          {
            subject: wrd.subject,
            word: wrd.word,
            sentence: outcome.sentence + ' ' + wrd.word,
            remaining: outcome.remaining.slice(wrd.word.length)
          }
        ]
        toPush[0].continued = generatePerfectOutcomes(toPush, completed)
        found.push(toPush)
      })
    })
    return found
  }

  function generateAllPossibleVariations(text) {
    let length = text.length
    let variations = []
    for (let begin = 0; begin < length; begin++) {
      let end = begin + 1
      while (end < length + 1) {
        variations.push(text.slice(begin, end))
        end++
      }
    }

    return variations
  }

  function findPossibilities(number) {
    let possibilities = []
    
    if (number == undefined || number == null) {
      return []
    }

    if (perfect) {
      number = number.replaceAll(' ', '')
      let completed = []

      let outcomes = generatePerfectOutcomes(
        [
          {
            subject: "",
            word: "",
            sentence: "",
            remaining: number,
            continued: []
          }
        ],
        completed
      )

      let converted = []

      completed.forEach(completion => {
        converted.push(completion.sentence)
      })

      return converted
    }

    // if there are spaces, check for words, not all variations
    if (number.indexOf(' ') >= 0) {
      let numberWords = number.split(' ')
      numberWords.forEach(word => {
        let possibility = ntw[word]
        if (possibility !== undefined) {
          possibilities.push(word + ' -> ' + possibility.join(', '))
        }
      })
      return possibilities
    } else {
      let variations = generateAllPossibleVariations(number.replace(' ', ''))

      variations.forEach(variation => {
        let possibility = ntw[variation]
        if (possibility !== undefined) {
          possibilities.push(variation + ' -> ' + possibility.join(', '))
        }
      })
      return possibilities
    }
  }

  $: possibilities = findPossibilities(number)

  // FOR SYNCING

  let sync

  $: {
    if (sync) {
      number = foundNumber
    }
  }
</script>

<div class="main">
  <div class="content">
    <h1>Words to Phone Number</h1>
    <p class="hint">This input takes words and converts them into numbers based on a phone's keypad.</p>
    <input type="text" bind:value={words} placeholder="enter words here" class="monospace">
    <p class="monospace results">{foundNumber == undefined ? '' : foundNumber}</p>
    <div class="options">
      <div class="monospace" id="sync-box"><input type="checkbox" bind:checked={sync}> sync mode</div>
      <div class="monospace" id="perfect-box"><input type="checkbox" bind:checked={perfect}> perfect matches (laggy)</div>
    </div>
  
    <h1>Phone Number to Words</h1>
    <p class="hint">This input takes numbers and tries to find words that match them based on the keypad's corresponding letters. If spaces are included, we look for matches on entire words only.</p>
    <p></p>
    <input type="text" bind:value={number} placeholder="enter numbers here" class="monospace"> 
    <p>{possibilities.length} results</p>
    <ul class="monospace results">
      {#each possibilities as possibility}
        <li class="monospace">
          {possibility}
        </li>
      {/each}
    </ul>
  </div>
</div>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&display=swap');
  @import url('https://fonts.googleapis.com/css2?family=Inter:wght@200;300;400;500;600;700;800&display=swap');
  
  * {
    font-family: 'Inter', sans-serif;
  }

  .main {
    width: 100%;
    height: 100%;
    padding: 0px;
    margin: 0px;

    display: flex;
    flex-direction: row;
    justify-content: center;
  }

  .content {
    width: min(700px, 100vw);
    margin: 40px;
  }

  .monospace {
    font-size: 16px;
    font-family: 'Space Mono', monoscape;
  }

  .results {
    margin-left: 2px;
    margin-top: 0;
    margin-bottom: 10px;
    background-color: rgba(143, 255, 57, 0.268);
  }

  .options {
    display: flex;
    flex-direction: row;
    justify-content: left;
    align-items: center;
  }

  #sync-box {
    display: flex;
    justify-content: left;
    align-items: center;
    background-color: rgba(2, 122, 149, 0.11);
    padding: 5px;
    margin: 0;
    width: 100%;
  }

  #perfect-box {
    display: flex;
    justify-content: left;
    align-items: center;
    background-color: rgba(149, 9, 2, 0.11);
    padding: 5px;
    margin: 0;
    width: 100%;
  }

  .hint {
    margin: 5px;
    padding: 10px 10px 10px 13px;
    background-color: rgba(0, 0, 0, 0.05);
  }

  input[type=checkbox] {
    font: inherit;
  }

  input[type=text] {
    margin-top: 10px;
    background-color: none;
    width: 100%;
    border: none;
    border-bottom: 2px solid rgb(64, 255, 0);
  }

  input[type=text]:focus {
    outline-width: 0;
  } 

  input[type=checkbox] {
    height: 20px;
    width: 20px;
    margin-right: 20px;
  }

  ul {
    padding-left: 30px;
    list-style-type: circle;
  }
</style>