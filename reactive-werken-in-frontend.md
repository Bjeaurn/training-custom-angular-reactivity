# Reactive werken in Frontend

Reactivity in software development is het "reageren" op verandering. Wanneer een waarde (om wat voor reden dan ook) veranderd, worden waardes die daarvan afhankelijk zijn ook bijgewerkt. 

Op een meer reactive manier programmeren stelt je in staat om ook "declaratief" te programmeren, waarbij je beschrijft "hoe" je wilt dat het veranderd, en veel minder "wat" er moet veranderen. Dit zorgt tevens voor een unidirectionele flow van data, dus die stroomt steeds maar 1 richting op. Dit heeft als voordeel dat het de leesbaarheid van dergelijke code een stuk minder complex maakt. Je hoeft immers enkel de syntax van de reactivity te begrijpen (dus hoe een signal of een RxJS stream werkt in Angular), en veel minder bezig te houden met waar alle draadjes heenlopen en op welke plaatsen er data gemuteerd wordt. 

De data stroomt van de ene naar de andere kant en beschrijft in de definitie exact hoe de waardes tot stand komen. Dat zorgt voor kleine behapbare blokken aan code die met een deel van de functionaliteit van doen hebben. Dat maakt het eenvoudig om op zichzelf te beoordelen en verhoogt de leesbaarheid, onderhoudbaarheid en uitbreidbaarheid van de code.


Beschouw het volgende voorbeeld: 

```ts
// Voorbeeld imperatieve class
export class Component extends OnInit {
    private service = inject(Service)
    todos: Todo[] = []

    ngOnInit() {
        this.service.getTodos().subscribe(data => this.todos = data.todos)
    }

    addTodo(todo: Todo) {
        this.todos.push(todo);
    }

    removeTodo(id: number) {
        const index = this.todos.findIndex(todo => todo.id === id)
        this.todos.splice(index, 1)
    }

    completeTodo(index: number) {
        this.todos[index].done = true
    }
}
```

Uiteraard is dit niet inherent fout of een slechte manier van werken. Er is een argument te maken dat dit uiterst eenvoudig is en goed leesbaar als gevolg. Toch is het mijn opinie dat in groter wordende en complexere systemen dit paradigma alleen zal zorgen voor onnodige complexiteit, veranderingen en uitbreiding tegengaan en er gemakkelijker fouten in kunnen sluipen. Het is tevens simpel om iets te vergeten als je alles met de hand bij moet werken. Laat mij het bovenstaande voorbeeld uitbreiden:

```ts
// Voorbeeld imperatieve class
export class Component extends OnInit {
    private service = inject(Service)
    todos: Todo[] = []
    allTodosCompleted: boolean = false // <-- Nieuw.

    ngOnInit() {
        this.service.getTodos().subscribe(data => this.todos = data.todos)
    }

    addTodo(todo: Todo) {
        this.todos.push(todo);
    }

    removeTodo(id: number) {
        const index = this.todos.findIndex(todo => todo.id === id)
        this.todos.splice(index, 1)
    }

    completeTodo(index: number) {
        this.todos[index].done = true

        // Nieuw stuk om allTodosCompleted te herberekenen...
        if(this.todos.every(todo => todo.done === true)) {
            this.allTodosCompleted = true
        } else {
            this.allTodosCompleted = false
        }
    }
}
```

Eenvoudig genoeg! En als je nu een set aan todos toevoegt en die een voor een op done zet, dan zal het netjes werken en is de waarde van `allTodosCompleted` correct. 
Maar wat als je nu een todo toevoegt? En wat voor andere soorten aan scenarios rondom todos kunnen we verzinnen waarin deze statussen kunnen veranderen?

```ts
// Voorbeeld imperatieve class
export class Component extends OnInit {
    private service = inject(Service)
    todos: Todo[] = []
    allTodosCompleted: boolean = false
    hideCompleted: boolean = true // <-- Nieuw.
    visibleTodos: Todo[] = [] // <-- Nieuw.


    ngOnInit() {
        this.service.getTodos().subscribe(data => this.todos = data.todos)
    }

    // Nieuwe functie
    calculateVisibleTodos() {
        this.visibleTodos = this.hideCompleted ? this.todos.filter(todo => todo.done !== true)) : this.todos
    }

    addTodo(todo: Todo) {
        this.todos.push(todo);
        this.calculateVisibleTodos() // <-- Toegevoegd.
    }

    removeTodo(id: number) {
        const index = this.todos.findIndex(todo => todo.id === id)
        this.todos.splice(index, 1)
        this.calculateVisibleTodos() // <-- Toegevoegd
    }

    completeTodo(index: number) {
        this.todos[index].done = true
        this.calculateVisibleTodos() // <-- Toegevoegd.

        if(this.todos.every(todo => todo.done === true)) {
            this.allTodosCompleted = true
        } else {
            this.allTodosCompleted = false
        }
    }
}
```
   
Misschien krijg je een idee waarom imperatief programmeren hier op termijn voor meer problemen kan zorgen dan het misschien initieel suggereert. Het handmatig moeten aanroepen van een functie om de status opnieuw te berekenen is niet alleen ineffecient, maar verwikkeld allerlei functies met andere functies. Het eerdere zo eenvoudige voorbeeld is nu hopeloos ingewikkeld aan het worden, waarbij je niet gemakkelijk kan bepalen of je de herbereken functie nu op alle plaatsen goed hebt staan, of er eventuele synchronisatie problemen kunnen ontstaan en zo verder. Tevens zijn de functies niet alleen maar verantwoordelijk voor hun stukje, maar krijgen ze meer verantwoordelijkheden.

Laten we het laatste voorbeeld nu in een modern reactive alternatief omzetten, waarbij het ook gelijk duidelijk wordt dat je veel meer declaratief kan programmeren.

```ts
// Reactive & Declarative
export class Component extends OnInit {
    private service = inject(Service)
    todos = signal<Todo[]>([])
    allTodosCompleted: boolean = computed(() => this.todos().every(todo => todo.done === true))
    hideCompleted = signal<boolean>(false)
    visibleTodos = computed(() => {
        if(this.hideCompleted()) {
            return this.todos().filter(todo => todo.completed !== true)
        }
        return this.todos()
    })

    ngOnInit() {
        this.service.getTodos().subscribe(data => this.todos.set(data.todos)) 
        // Laten we zo voor eenvoud, maar dit kan ook declaratiever middels correct RxJS gebruik! Minder manueel subscriben is beter!
        // Een beter voorbeeld zou zijn om een `toSignal()` te gebruiken, of als je RxJS meer toepassingen heeft, deze te assignen aan een `data$` en dit in de template middels een `| async` te subscriben. Zo is het framework volledig in controle over jouw subscription management. Geen handmatige controles of we niet vergeten te unsubscriben!
    }

    addTodo(todo: Todo) {
        // Immutability helpt bij reactivity om nieuwe waarden te herkennen. Heeft de voorkeur over "inner-mutatie" waar mogelijk!
        this.todos.update(prev => [...prev, todo])
    }

    removeTodo(id: number) {
        this.todos.update(todos => {
            const index = todos.findIndex(todo => todo.id === id)
            const removed = todos.splice(index, 1);
            return todos
        })
    }

    completeTodo(id: number) {
        const index = this.todos().findIndex(todo => todo.id === id)
        this.todos().map(todo => todo.id === id).done = true
    }
}
```

In dit voorbeeld zijn vrijwel alle waardes reactive, staat er bij hun declaratie beschreven hoe zij aan hun waarden komen, en zijn verdere functies (in theorie) eenvoudige manipulaties van bestaande data stromen. Ook hoeven de functies geen aanvullende verantwoordelijkheden te dragen, zoals het correct triggeren van herberekeningen.
Door de data op deze manier zo veel mogelijk uniform door de applicatie te laten stromen is het eenvoudig (en leesbaar!) om dit later verder uit te breiden met nieuwe stromen die inpluggen op bestaande. 
Daarnaast is het niet mogelijk om ergens een functie te vergeten, waardoor je minder gauw fouten maakt of iets over het hoofd ziet. 
De code is zo veel leesbaarder, heeft minder gedeelde verantwoordelijkheden en dat maakt op termijn voor een betere en onderhoudbaardere applicatie.
