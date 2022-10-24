<script>
  let number = ''

  import ntw from '$lib/numbers-to-words.json'

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

    let variations = generateAllPossibleVariations(number.replace(' ', ''))

    variations.forEach(variation => {
      let possibility = ntw[variation]
      if (possibility !== undefined) {
        possibilities.push(variation + ' -> ' + possibility.join(', '))
      }
    })
    return possibilities
  }

  $: possibilities = findPossibilities(number)
</script>

<h1>Phone Number to Words</h1>
<h3>By <a href="https://finn.so">Finn Scicluna-O'Prey</a></h3>
<input type="text" bind:value={number} placeholder="enter your number">
<ul>
	{#each possibilities as possibility}
		<li>
			{possibility}
    </li>
	{/each}
</ul>

