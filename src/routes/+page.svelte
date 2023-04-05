<script>
    import {onMount} from 'svelte';

    let questions = [];
    let currentQuestion;
    let score = 0;
    let answered = 0;
    let difficulty = "easy";
    let category = 9; // General Knowledge
    let isLoading = true;
    let categories = [];


    async function fetchCategories() {
        const response = await fetch('https://opentdb.com/api_category.php');
        const data = await response.json();
        return data.trivia_categories;
    }


    const fetchQuestions = async () => {
        isLoading = true;
        const response = await fetch(`https://opentdb.com/api.php?amount=10&category=${category}&difficulty=${difficulty}&type=multiple`);
        const data = await response.json();
        questions = data.results.map((q) => {
            return {
                ...q,
                answers: [...q.incorrect_answers, q.correct_answer].sort(() => Math.random() - 0.5),
            };
        });
        currentQuestion = questions[0];
        isLoading = false;
    };

    const submitAnswer = (selectedAnswer) => {
        if (selectedAnswer === currentQuestion.correct_answer) {
            score++;
        }
        answered++;
        if (answered < questions.length) {
            currentQuestion = questions[answered];
        } else {
            alert(`Game over! Your score is ${score}/${answered}.`);
            score = 0;
            answered = 0;
            fetchQuestions();
        }
    };

    const decodeHtml = (html) => {
        const txt = document.createElement("textarea");
        txt.innerHTML = html;
        return txt.value;
    };



    onMount(async () => {
        categories = await fetchCategories();
        fetchQuestions();
    });

</script>

<section class="section">
    <div class="container">
        <h1 class="title">Trivia App</h1>

        {#if isLoading}
            <p>Loading...</p>
        {:else}
            <div class="box">
                <h2 class="subtitle">{decodeHtml(currentQuestion.question)}</h2>
                <ul class="columns is-multiline">
                    {#each currentQuestion.answers as answer}
                        <li class="column is-one-quarter">
                            <button class="button is-primary is-fullwidth" on:click={() => submitAnswer(answer)}>{decodeHtml(answer)}</button>
                        </li>
                    {/each}
                </ul>
                <p>Score: {score}/{answered}</p>
            </div>
        {/if}

        <div class="field">
            <label class="label">
            Difficulty:
                <div class="control">
                    <div class="select">
                        <select bind:value="{difficulty}" on:change="{fetchQuestions}">
                        <option value="easy">Easy</option>
                        <option value="medium">Medium</option>
                        <option value="hard">Hard</option>
                    </select>
                    </div>
                </div>
        </label>
        </div>

        <div class="field">
            <label class="label">
            Category:
                <div class="control">
                    <div class="select">
                        <select bind:value="{category}" on:change="{fetchQuestions}">
                        {#each categories as cat}
                            <option value="{cat.id}">{cat.name}</option>
                        {/each}
                    </select>
                    </div>
                </div>
        </label>
        </div>
    </div>
</section>

