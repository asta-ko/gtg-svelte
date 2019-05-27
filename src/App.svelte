<style>
/* 0 */
@import 'main.css';
</style>
<script>
import { writable } from 'svelte/store';

function init_db() {
    const low = require('lowdb')
    const LocalStorage = require('lowdb/adapters/LocalStorage')

    const adapter = new LocalStorage('db')
    const db = low(adapter)

    db.defaults({ answers: [], count: 0 })
        .write()
    return db
}

function get_count(db) {
    return db.get('answers')
        .size()
        .value()
}

function save_answers(db, answers) {
    let answers1 = answers.flat(2);
    db.get('answers')
        .push({ ans: answers1, date: new Date() })
        .write()
}

function get_answers(db) {
    return db.get('answers')
        .map('ans')
        .value();
}

function round(num) {
    return Math.round(num * 10) / 10
}

function get_percents(answers) {

    let a = answers.reduce((r, a) => a.map((b, i) => (r[i] || 0) + b), []);

    function perc_arr(arr) {
        let sum = arr.reduce((a, b) => a + b, 0)
        let perc_a = arr.map((x) => round(x * 100 / sum) + '%', 0)
        return perc_a
    }


    let percents1 = perc_arr(a.slice(0, 2))
    let percents2 = perc_arr(a.slice(2, 6))
    let percents3 = perc_arr(a.slice(6, 9))
    let percents4 = perc_arr(a.slice(9, 13))
    let percents5 = perc_arr(a.slice(13, 19))
    return [percents1, percents2, percents3, percents4, percents5]

}

function get_max_percents(percent_arr) {
    let num_arr = percent_arr.map((x) => parseFloat(x))
    return Math.max(...num_arr) + '%'
}

function max_percents_arr(arr) {
    let perc = get_percents(arr);
    let max_percents = perc.map((x) => get_max_percents(x))
    return max_percents
}

function plain_arr(arr) {
    let a = arr.reduce((r, a) => a.map((b, i) => (r[i] || 0) + b), []);
    let percents1 = a.slice(0, 2)
    let percents2 = a.slice(2, 6)
    let percents3 = a.slice(6, 9)
    let percents4 = a.slice(9, 13)
    let percents5 = a.slice(13, 19)
    return [percents1, percents2, percents3, percents4, percents5]
}


function next_page() {

    clearTimeout(timeout)

    let clean_answers = [
        [0, 0],
        [0, 0, 0, 0],
        [0, 0, 0],
        [undefined, undefined, undefined, undefined],
        [0, 0, 0, 0, 0, 0]
    ]

    let clean = Array.from(clean_answers);

    if ($answers == []) {
        answers.set(clean)
    }

    if (page == 1) {
        answers.update(n => clean)
    }

    if (page == 7) {
        page = 1;

    } else {
        page += 1;
        let seconds = 20000
        if (page == 7) {
        	seconds = 30000
        }

        timeout = setTimeout(() => {
            page = 1
        }, seconds);
    }
}



$: db = init_db()

$: page = 1;

let answers = writable([]);

$: ans = $answers

$: percents = get_percents(get_answers(db))

$: plain = plain_arr(get_answers(db))

$: max = max_percents_arr(get_answers(db))

$: count = get_count(db)

$: timeout = undefined;


window.oncontextmenu = function(event) {
     event.preventDefault();
     event.stopPropagation();
     return false;
};


</script>
<div class="main page{page}">
    <div class="header">
        <div class="header_one">Культурно-образовательные и музейные комплексы</div>
        <div class="header_two">Владивосток / Калининград / Кемерово / Севастополь</div>
        {#if page != 1 && page != 7}
        <div class="return" on:click={()=> {page = 1}}>начать заново</div>
        {/if}
    </div>
    {#if page == 1}
    <div class="text"><p>Концепции комплексов для каждого региона сейчас активно разрабатываются и обсуждаются.</p> <p>Присоединитесь к диалогу, чтобы обсудить значение новых комплексов для регионов, сообществ и людей</p></div>
    <div class="buttons"><button class="button" on:click={()=> {next_page();}}>ПОДЕЛИТЕСЬ МНЕНИЕМ</button></div>
    {/if}
    {#if page == 2}
    <div class="text"></div>
    <div class="buttons"> <button class="button" on:click={()=> {next_page(); $answers[0][0] = 1;}}>РАБОТАЮ В СФЕРЕ КУЛЬТУРЫ</button><button class="button" on:click={()=> {next_page(); $answers[0][1] = 1;}}>ПОСЕТИТЕЛЬ ФЕСТИВАЛЯ</button></div>
    {/if}
    {#if page == 3}
    <div class="page">2/5</div>
    <div class="text">Новые комплексы смогут повлиять на развитие регионов:</div>
    <div class="buttons"><button class="button" on:click={()=> {next_page(); $answers[1][0] = 1}}>ПОЗИТИВНО</button><button class="button" on:click={()=> {next_page(); $answers[1][1] = 1}}>НЕГАТИВНО</button><button class="button" on:click={()=> {next_page(); $answers[1][2] = 1}}>НЕ ПОВЛИЯЮТ</button><button class="button" on:click={()=> {next_page(); $answers[1][3] = 1}}>НЕ ЗНАЮ</button></div>
    {/if}
    {#if page == 4}
    <div class="page">3/5</div>
    <div class="text">В чем измерять успех и эффективность культурных комплексов и музеев, запланированных в них?</div>
    <div class="buttons"><button class="button" style="height:96px;" on:click={()=> {next_page(); $answers[2][0] = 1}}>ВАЖНЫ КОЛИЧЕСТВЕННЫЕ ПОКАЗАТЕЛИ (ПОСЕЩАЕМОСТЬ, МЕРОПРИЯТИЯ)</button><button class="button" style="height: 160.22px;" on:click={()=> {next_page(); $answers[2][1] = 1}}>ПРИОРИТЕТ ЗА КАЧЕСТВЕННЫМИ ПОКАЗАТЕЛЯМИ (КАЧЕСТВО ЖИЗНИ, ПРЕСТИЖНОСТЬ РЕГИОНОВ, РАЗВИТИЕ ЛОКАЛЬНОГО И НАЦИОНАЛЬНОГО САМОСОЗНАНИЯ, РАЗВИТИЕ ТВОРЧЕСКОГО ПОТЕНЦИАЛА ЛИЧНОСТИ И Т.Д</button><button class="button" on:click={()=> {next_page(); $answers[2][2] = 1}}>НЕ ЗНАЮ</button></div>
    {/if}
    {#if page == 5}
    <div class="page">4/5</div>
    <div class="text">Считаете ли Вы, что новые комплексы:</div>
    <div class="buttons">
        <div class="divTable">
            <div class="divTableBody">
                <div class="divTableRow">
                    <div class="divTableCell tc1"><span>смогут стать культурными символами регионов и брендами территорий</span></div>
                    <div class="divTableCell tc2"><button class='smallbutton {$answers[3][0] ? "yes":""}' on:click={()=> {$answers[3][0] = 1}}>ДА</button><button class="smallbutton {$answers[3][0] === 0 ? 'no':''}" on:click={()=> {$answers[3][0] = 0}}>НЕТ</button></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell tc1"><span>будут способствовать увеличению туристических потоков в регионах, положительно влиять на демографию и занятость населения в регионах</span></div>
                    <div class="divTableCell tc2"><button on:click={()=> {$answers[3][1] = 1}} class="smallbutton {$answers[3][1] ? 'yes':''}">ДА</button><button class="smallbutton {$answers[3][1] === 0 ? 'no':''}" on:click={()=> {$answers[3][1] = 0}}>НЕТ</button></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell tc1"><span>должны учитывать локальный контекст в своих программах</span></div>
                    <div class="divTableCell tc2"><button class="smallbutton {$answers[3][2] ? 'yes':''}" on:click={()=> {$answers[3][2] = 1}}>ДА</button><button class="smallbutton {$answers[3][2] === 0 ? 'no':''}" on:click={()=> {$answers[3][2] = 0}}>НЕТ</button></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell tc1"><span>интерес к местным музеям снизится с открытием новых комплексов</span></div>
                    <div class="divTableCell tc2"><button class="smallbutton {$answers[3][3] ? 'yes':''}" on:click={()=> {$answers[3][3] = 1}}>ДА</button><button class="smallbutton {$answers[3][3] === 0 ? 'no':''}" on:click={()=> {$answers[3][3] = 0}}>НЕТ</button></div>
                </div>
            </div>
        </div>
    </div>
    <button class="sendbutton button" disabled={$answers[3].includes(undefined)} on:click={()=> next_page()}>ОТВЕТИТЬ</button>
    {/if}
    {#if page == 6}
    <div class="page">5/5</div>
    <div class="text">За что Вы любите какой-либо музей?</div>
    <div class="buttons">
        <div class="divTable">
            <div class="divTableBody">
                <div class="divTableRow">
                    <div class="divTableCell">
                        <div class="checkbox {$answers[4][0] ? 'checked':''}" on:click={()=> {$answers[4][0] = +!$answers[4][0]}}></div>
                    </div>
                    <div class="divTableCell tc3"><span>Возможность узнавать новое</span></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell">
                        <div class="checkbox {$answers[4][1] ? 'checked':''}" on:click={()=> {$answers[4][1] = +!$answers[4][1]}}></div>
                    </div>
                    <div class="divTableCell tc3"><span>Мы собираемся с друзьями и близкими, чтобы провести время</span></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell">
                        <div class="checkbox {$answers[4][2] ? 'checked':''}" on:click={()=> {$answers[4][2] = +!$answers[4][2]}}></div>
                    </div>
                    <div class="divTableCell tc3"><span>Заряжаюсь эмоциями и новыми впечатлениями</span></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell">
                        <div class="checkbox {$answers[4][3] ? 'checked':''}" on:click={()=> {$answers[4][3] = +!$answers[4][3]}}></div>
                    </div>
                    <div class="divTableCell tc3"><span>Интересно встречать новых людей</span></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell">
                        <div class="checkbox {$answers[4][4] ? 'checked':''}" on:click={()=> {$answers[4][4] = +!$answers[4][4]}}></div>
                    </div>
                    <div class="divTableCell tc3"><span>Другое</span></div>
                </div>
                <div class="divTableRow">
                    <div class="divTableCell">
                        <div class="checkbox {$answers[4][5] ? 'checked':''}" on:click={()=> {$answers[4][5] = +!$answers[4][5]}}></div>
                    </div>
                    <div class="divTableCell tc3"><span>Не знаю</span></div>
                </div>
            </div>
        </div>
    </div>
    <button class="sendbutton button" disabled={!$answers[4].includes(1)} on:click={()=> {save_answers(db, $answers); percents = get_percents(get_answers(db)); count = get_count(db); max = max_percents_arr(get_answers(db)); plain = plain_arr(get_answers(db)); next_page();}}>ОТВЕТИТЬ</button>
    {/if}
    {#if page == 7}
    <div class="text">Спасибо за участие!</div>
    <div class="seven_a">
        <h3>Пара слов о себе</h3>
        <p>работаю в сфере культуры — <span class="{percents[0][0] == max[0] ? 'bold':''}">{percents[0][0]}</span></p>
        <p>посетитель фестиваля — <span class="{percents[0][1] == max[0] ? 'bold':''}">{percents[0][1]}<span></p>
        <h3>Новые комплексы смогут повлиять на развитие регионов:</h3>
        <p>позитивно — <span class="{percents[1][0] == max[1] ? 'bold':''}">{percents[1][0]}</span></p>
        <p>негативно — <span class="{percents[1][1] == max[1] ? 'bold':''}">{percents[1][1]}</span></p>
        <p>не повлияют — <span class="{percents[1][2] == max[1] ? 'bold':''}">{percents[1][2]}</span></p>
        <p>не знаю — <span class="{percents[1][3] == max[1] ? 'bold':''}">{percents[1][3]}</span></p>
        <h3>В чем измерять успех и эффективность культурных комплексов и музеев, запланированных в них?</h3>
        <p>Важны количественные показатели — <span class="{percents[2][0] == max[2] ? 'bold':''}">{percents[2][0]}</span></p>
        <p>Приоритет за качественными показателями — <span class="{percents[2][1] == max[2] ? 'bold':''}">{percents[2][1]}</span></p>
        <p>Не знаю — <span class="{percents[2][2] == max[2] ? 'bold':''}">{percents[2][2]}</span></p>
    </div>
    <div class="seven_b">
        <h3>Считаете ли Вы, что новые комплексы:</h3>
        <p>смогут стать культурными символами регионов и брендами территорий — <span class="{plain[3][0] >= count/2 ? 'bold':''}">да</span>/<span class="{plain[3][0] < count/2 ? 'bold':''}">нет</span></p>
        <p>будут способствовать увеличению туристических потоков в регионах,</p>
        <p>положительно влиять на демографию и занятость населения в регионах — <span class="{plain[3][1] >= count/2 ? 'bold':''}">да</span>/<span class="{plain[3][1] < count/2 ? 'bold':''}">нет</span></p>
        <p>должны учитывать локальный контекст в своих программах — <span class="{plain[3][2] >= count/2 ? 'bold':''}">да</span>/<span class="{plain[3][2] < count/2 ? 'bold':''}">нет</span></p>
        <p>Интерес к местным музеям снизится с открытием новых комплексов —<span class="{plain[3][3] >= count/2 ? 'bold':''}">да</span>/<span class="{plain[3][3] < count/2 ? 'bold':''}">нет</span></p>
        <h3>За что Вы любите какой-либо музей?</h3>
        <p>Возможность узнавать новое — <span class="{percents[4][0] == max[4] ? 'bold':''}">{percents[4][0]}</span></p>
        <p>Мы собираемся с друзьями и близкими, чтобы провести время — <span class="{percents[4][1] == max[4] ? 'bold':''}">{percents[4][1]}</span></p>
        <p>Заряжаюсь эмоциями и новыми впечатлениями — <span class="{percents[4][2] == max[4] ? 'bold':''}">{percents[4][2]}</span></p>
        <p>Интересно встречать новых людей — <span class="{percents[4][3] == max[4] ? 'bold':''}">{percents[4][3]}</span></p>
        <p>Другое — <span class="{percents[4][4] == max[4] ? 'bold':''}">{percents[4][4]}</span></p>
        <p>Не знаю — <span class="{percents[4][5] == max[4] ? 'bold':''}">{percents[4][5]}</span></p>
        <h3>Ответили {count} человек</h3>
    </div>
    <button class="sendbutton button" on:click={()=> { next_page();}}>ЗАВЕРШИТЬ</button>
    {/if}
</div>