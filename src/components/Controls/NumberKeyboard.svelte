<script>
	import { userGrid } from '@sudoku/stores/grid';
	import { keyboardDisabled } from '@sudoku/stores/keyboard';
	import { cursor } from '@sudoku/stores/cursor';
	import { notes } from '@sudoku/stores/notes';
	import { candidates } from '@sudoku/stores/candidates';
	import { StackManager, SetValueCommand, ResetCommand, SetNoteCommand, copyUserGrid, copyCandidates, errorValue } from './ActionBar/Resetstack'
	import { strategyManager } from '@sudoku/sudokuStrategies/StrategyManager'

    export let number;
    $: disabled = false;
    let showNoSolutionAlert = false;

	// 检查输入是否合法
	function isValidInput(grid, x, y, num) {
		// 检查行
		for(let i = 0; i < 9; i++) {
			if(i !== x && grid[y][i] === num) return false;
		}
		// 检查列
		for(let i = 0; i < 9; i++) {
			if(i !== y && grid[i][x] === num) return false;
		}
		// 检查3x3方格
		let boxX = Math.floor(x/3) * 3;
		let boxY = Math.floor(y/3) * 3;
		for(let i = boxY; i < boxY + 3; i++) {
			for(let j = boxX; j < boxX + 3; j++) {
				if((i !== y || j !== x) && grid[i][j] === num) return false;
			}
		}
		return true;
	}

	function handleKeyButton(num) {
		if (!$keyboardDisabled) {
			if ($notes) {
				StackManager.pushCmd(new SetNoteCommand(copyUserGrid($userGrid), copyCandidates($candidates), num, {'x': $cursor.x, 'y': $cursor.y}));
				if (num === 0) {
					candidates.clear($cursor);
				} else {
					candidates.add($cursor, num);
				}
				userGrid.set($cursor, 0);
			} else {
				// 多候选值输入，启用回溯功能
				if ($candidates.hasOwnProperty($cursor.x + ',' + $cursor.y) && $candidates[$cursor.x + ',' + $cursor.y].length > 1) {
					StackManager.pushCmd(new ResetCommand(copyUserGrid($userGrid), copyCandidates($candidates), num, {'x': $cursor.x, 'y': $cursor.y}));
				} else {
					StackManager.pushCmd(new SetValueCommand(copyUserGrid($userGrid), copyCandidates($candidates), num, {'x': $cursor.x, 'y': $cursor.y}));
				}

				if ($candidates.hasOwnProperty($cursor.x + ',' + $cursor.y)) {
					candidates.clear($cursor);
				}

				userGrid.set($cursor, num);

				// 只在栈不为空且输入合法时检查是否有解
				if (StackManager.canUndo() && isValidInput($userGrid, $cursor.x, $cursor.y, num)) {
					let candidatesList = strategyManager.applyStrategies($userGrid);
					let hasEmptyCell = false;
					let hasSolution = true;

					for(let row = 0; row < 9 && hasSolution; row++) {
						for(let col = 0; col < 9; col++) {
							if($userGrid[row][col] === 0) {
								hasEmptyCell = true;
								if(candidatesList[row][col].length === 0) {
									hasSolution = false;
									break;
								}
							}
						}
					}

					if(hasEmptyCell && !hasSolution) {
						// 记录导致无解的数字和位置
						errorValue.number = num;
						errorValue.index = {x: $cursor.x, y: $cursor.y};
						showNoSolutionAlert = true;
						setTimeout(() => {
							showNoSolutionAlert = false;
							// 清除错误标记
							errorValue.number = undefined;
							errorValue.index = {};
						}, 3000);
					}
				}
			}
		}
	}
</script>

<button class="btn btn-key" disabled={$keyboardDisabled || disabled} title="Insert {number}" on:click={() => handleKeyButton(number)}>
    {number}
</button>

{#if showNoSolutionAlert}
	<div class="alert-overlay">
		<div class="alert-box">
			<p>No solution found!</p>
		</div>
	</div>
{/if}

<style>
	.alert-overlay {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: 1000;
	}
	
	.alert-box {
		background-color: #fff;
		border: 2px solid #ef4444;
		border-radius: 0.5rem;
		padding: 1rem 2rem;
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
		animation: fadeIn 0.3s ease-out;
	}
	
	.alert-box p {
		color: #ef4444;
		font-weight: 600;
		margin: 0;
	}
	
	@keyframes fadeIn {
		from {
			opacity: 0;
			transform: translateY(-10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
</style>