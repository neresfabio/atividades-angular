## Criando To-Do-list com Angular

Olá! Que ótimo que você está começando com Angular e já tem um projeto prático em mente. Uma to-do-list é um excelente exercício para você colocar em prática conceitos fundamentais do Angular, como componentes, data binding, diretivas e comunicação com serviços – e, depois, para integrar com sua API em Spring Boot.

A seguir, vou te passar um roteiro de atividade que vai te ajudar a desenvolver esse projeto de forma prática, passo a passo.

---

1. Iniciando o Projeto Angular

a) Instale o Angular CLI (caso ainda não o tenha instalado):

   `npm install -g @angular/cli`

b) Crie um novo projeto:

   `ng new todo-list-app`

Durante a criação, você pode optar por incluir roteamento e escolher o estilo (CSS, SCSS, etc.) conforme sua preferência.

2. Criando Componentes e Estrutura Básica

a) Crie um componente para a lista de tarefas:

   `ng generate component components/todo-list`

b) Crie outro para o item da tarefa, se achar interessante modularizar:

   `ng generate component components/todo-item`

Você pode também criar um serviço para gerenciar as tarefas localmente (antes de integrá-lo à API):

   `ng generate service services/todo`

3. Implementando a Lógica da To-Do List

No serviço (src/app/services/todo.service.ts), defina uma estrutura simples para armazenar os itens da to-do list:

```
import { Injectable } from '@angular/core';

export interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TodoService {
  private todos: Todo[] = [];
  private nextId = 1; // para gerar IDs simples

  constructor() { }

  getTodos(): Todo[] {
    return this.todos;
  }

  addTodo(title: string): void {
    this.todos.push({
      id: this.nextId++,
      title,
      completed: false
    });
  }

  toggleTodo(id: number): void {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }

  removeTodo(id: number): void {
    this.todos = this.todos.filter(t => t.id !== id);
  }
}

```

No componente todo-list (src/app/components/todo-list/todo-list.component.ts), injete o serviço e interaja com ele:

```
import { Component, OnInit } from '@angular/core';
import { TodoService, Todo } from '../../services/todo.service';

@Component({
  selector: 'app-todo-list',
  templateUrl: './todo-list.component.html'
})
export class TodoListComponent implements OnInit {
  todos: Todo[] = [];
  newTodoTitle: string = '';

  constructor(private todoService: TodoService) { }

  ngOnInit(): void {
    this.todos = this.todoService.getTodos();
  }

  addTodo(): void {
    if (this.newTodoTitle.trim()) {
      this.todoService.addTodo(this.newTodoTitle);
      this.newTodoTitle = '';
      this.todos = this.todoService.getTodos(); // atualiza a lista
    }
  }

  toggleTodo(todo: Todo): void {
    this.todoService.toggleTodo(todo.id);
  }

  removeTodo(todo: Todo): void {
    this.todoService.removeTodo(todo.id);
    this.todos = this.todoService.getTodos();
  }
}

```

E no HTML (src/app/components/todo-list/todo-list.component.html) você pode ter algo como:

```
<div class="todo-container">
  <h1>Minha To-Do List</h1>
  <div class="input-group">
    <input [(ngModel)]="newTodoTitle" placeholder="Nova tarefa" />
    <button (click)="addTodo()">Adicionar</button>
  </div>

  <ul>
    <li *ngFor="let todo of todos">
      <span (click)="toggleTodo(todo)" [ngClass]="{'completed': todo.completed}">
        {{ todo.title }}
      </span>
      <button (click)="removeTodo(todo)">Remover</button>
    </li>
  </ul>
</div>

```

Não se esqueça de importar FormsModule no seu módulo principal (src/app/app.module.ts) para que o ngModel funcione:

```
import { FormsModule } from '@angular/forms';

@NgModule({
  declarations: [ /* seus componentes */ ],
  imports: [
    BrowserModule,
    FormsModule, // adicione aqui
    // outros módulos
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

4. Estilizando um Pouco

Você pode adicionar uma classe CSS para visualizar quando uma tarefa estiver concluída. Por exemplo, no seu CSS (src/app/components/todo-list/todo-list.component.css):

```
.completed {
  text-decoration: line-through;
  color: gray;
}
```

5. Integração com a API Spring Boot (Em Um Segundo Momento)

Depois que sua aplicação estiver funcionando com dados locais, você pode migrar para consumir a API que criou. Aqui vai um resumo dos passos:

a) Crie um serviço para integrar com a API, por exemplo, TodoHttpService.
   Use o HttpClient para fazer requisições REST. Não se esqueça de importar HttpClientModule no seu AppModule.

```
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

export interface Todo {
  id: number;
  title: string;
  completed: boolean;
}

@Injectable({
  providedIn: 'root'
})
export class TodoHttpService {

  private apiUrl = 'http://localhost:8080/api/todos';  // ajuste conforme sua API

  constructor(private http: HttpClient) { }

  getTodos(): Observable<Todo[]> {
    return this.http.get<Todo[]>(this.apiUrl);
  }

  addTodo(todo: Partial<Todo>): Observable<Todo> {
    return this.http.post<Todo>(this.apiUrl, todo);
  }

  updateTodo(todo: Todo): Observable<Todo> {
    return this.http.put<Todo>(`${this.apiUrl}/${todo.id}`, todo);
  }

  deleteTodo(id: number): Observable<void> {
    return this.http.delete<void>(`${this.apiUrl}/${id}`);
  }
}
```

b) Atualize o seu componente para usar esse serviço. Na fase de migração, você precisará refatorar o código para trabalhar com Observables, utilizando o subscribe.

6. Dicas Práticas para Aprender Fazendo

• Comece com o básico utilizando dados locais. Entenda como funcionam os componentes, data binding, diretivas e serviços.
• Teste cada funcionalidade – adição, remoção e atualização – de forma independente.
• Gradualmente, substitua as funções do TodoService por chamadas reais à API, cuidando de lidar com assincronismo.
• Utilize ferramentas de depuração (como o console do navegador) para acompanhar o fluxo de dados.
• Busque, se possível, implementar mensagens de erro e feedback para o usuário, melhorando a experiência.

Considerações Finais

Durante o desenvolvimento, não hesite em fazer perguntas e buscar apoio na comunidade Angular (fóruns, Stack Overflow e grupos especializados). Com o tempo, ao integrar a API, você também vai ganhar experiência na comunicação entre front-end e back-end.

Espero que esse roteiro te ajude a colocar a mão na massa! Se você tiver dúvidas durante o desenvolvimento ou precisar de mais detalhes sobre alguma parte específica (por exemplo, configuração do HttpClient ou dicas de testes), só avisar. Bom código e sucesso na sua jornada com Angular!

---
   
1. Organize sua aplicação em pastas para cada camada:  
   Uma abordagem comum é criar pastas como:  
   • components – para os componentes de apresentação.  
   • services – para os serviços que fazem a ponte com a camada de negócio ou dados.  
   • models ou domain – para as classes e interfaces que representam suas entidades.  
   • data ou repositories – para implementações de acesso a dados (se for o caso).  
   • modules – para agrupamentos de componentes e seus serviços que formam funcionalidades do app.

2. Excluir ou renomear arquivos padrão:  
   Se você deseja começar a sua aplicação organizando em camadas, pode fazer o seguinte:  
   • Excluir ou renomear os arquivos default do app.component (app.component.ts, app.component.html, app.component.css e app.component.spec.ts) se eles não se encaixarem na sua nova estrutura.  
   • No app.module.ts, remova a referência ao AppComponent (ou realoque-o para se encaixar na sua arquitetura). Por exemplo, se você criar um módulo chamado CoreModule ou MainModule, pode importar esse novo módulo no AppModule e mover a estrutura para lá.
   
3. Crie os seus módulos:  
   • Uma boa prática é ter um módulo Core para serviços globais e talvez um módulo Shared para componentes reutilizáveis.  
   • Se preferir, crie um módulo específico para cada camada/funcionalidade e importe-os no módulo raiz.

O importante é manter a clareza e o gerenciamento lógico das dependências entre as camadas. Dê uma olhada em arquiteturas como "feature modules" e "core/shared modules" para se inspirar na organização da sua aplicação. Dessa forma, você terá um projeto "limpo" e estruturado conforme a sua visão de camadas.
