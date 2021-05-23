<script lang="ts">
  import Todos from './components/Todos.svelte'
	import Alert from './components/Alert.svelte'

	import { todos } from './stores'
	const ms = 2000
</script>
<Alert ms={ms}/>
<Todos bind:todos={$todos}/>