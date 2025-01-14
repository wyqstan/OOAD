<script>
import { strategyShow, strategyDisplay } from '@sudoku/stores/strategyShow'

</script>

<div class="character-display">
	{#if $strategyShow}
		<p>{""}</p>
    {:else}
        <p>{$strategyDisplay}</p>
    {/if}

</div>

<style global>
	.character-display {
    		display: flex;
    		justify-content: center; /* 水平方向居中 */
    		height: 50px;          /* 设置容器高度，根据需要调整 */
    		text-align: center;
    		margin: 1rem 0;
    		font-size: 1rem;
    		color: #333;
    		background-color: #f9f9f9;
    		padding: 0.5rem;
    		border-radius: 8px;
    		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    	}
</style>