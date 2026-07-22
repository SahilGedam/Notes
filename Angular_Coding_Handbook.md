# Angular Interview Coding Handbook (Angular 20 & Node.js/Express)

This handbook is designed for frontend developers preparing for a 30-minute coding interview at firms like Deloitte, Accenture, Capgemini, Cognizant, LTIMindtree, etc. It provides executable, modern Angular 20 (Standalone Components, Signals, RxJS, functional structures) and Node.js/Express patterns.

---

## SECTION 1: Component Communication (Basic)

### Question
> Create a Parent component and a Child component. Pass an object named `user` and an array named `hobbies` from Parent to Child using `@Input`. Send a string notification back from Child to Parent on a button click using `@Output` and `EventEmitter`.

### Difficulty
Easy

### Concepts Tested
* `@Input` and `@Output` decorator properties
* `EventEmitter`
* Property and Event Binding
* Standalone Components

### Folder Structure
```text
src/
  app/
    components/
      parent/
        parent.component.ts
        parent.component.html
      child/
        child.component.ts
        child.component.html
```

### Complete Code

#### `src/app/components/child/child.component.ts`
```typescript
import { Component, Input, Output, EventEmitter } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-child',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './child.component.html',
  styleUrls: []
})
export class ChildComponent {
  @Input() user!: { name: string; role: string };
  @Input() hobbies: string[] = [];
  @Output() statusChanged = new EventEmitter<string>();

  sendNotification() {
    this.statusChanged.emit('Child action completed successfully!');
  }
}
```

#### `src/app/components/child/child.component.html`
```html
<div style="border: 2px dashed #ccc; padding: 15px; margin-top: 10px; border-radius: 6px;">
  <h4>Child Component</h4>
  <p><strong>Name:</strong> {{ user.name }}</p>
  <p><strong>Role:</strong> {{ user.role }}</p>
  <h5>Hobbies:</h5>
  <ul>
    <li *ngFor="let hobby of hobbies">{{ hobby }}</li>
  </ul>
  <button (click)="sendNotification()" style="background-color: #0076d6; color: white; border: none; padding: 8px 12px; border-radius: 4px; cursor: pointer;">
    Notify Parent
  </button>
</div>
```

#### `src/app/components/parent/parent.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { ChildComponent } from '../child/child.component';

@Component({
  selector: 'app-parent',
  standalone: true,
  imports: [CommonModule, ChildComponent],
  templateUrl: './parent.component.html',
  styleUrls: []
})
export class ParentComponent {
  parentUser = { name: 'Sahil Gedam', role: 'Frontend Architect' };
  parentHobbies = ['Coding', 'Reading', 'Gaming'];
  receivedMessage = '';

  handleChildNotification(message: string) {
    this.receivedMessage = message;
  }
}
```

#### `src/app/components/parent/parent.component.html`
```html
<div style="border: 2px solid #0076d6; padding: 20px; border-radius: 8px; max-width: 500px; margin: 20px auto; font-family: sans-serif;">
  <h3>Parent Component</h3>
  <p *ngIf="receivedMessage" style="background-color: #e6f7ff; color: #0050b3; padding: 10px; border-radius: 4px;">
    <strong>Message from Child:</strong> {{ receivedMessage }}
  </p>
  
  <app-child 
    [user]="parentUser" 
    [hobbies]="parentHobbies" 
    (statusChanged)="handleChildNotification($event)">
  </app-child>
</div>
```

### Step-by-step Explanation
1. **Standalone Components**: `standalone: true` configures components without needing a parent NgModule. We explicitly import `CommonModule` to use structural directives (`*ngFor`, `*ngIf`).
2. **`@Input` & `@Output`**: Parent binds parameters to `[user]` and `[hobbies]`. The child accepts them via `@Input()`.
3. **EventEmitter**: When the child's button is clicked, `statusChanged.emit()` fires an event, which the parent intercepts via `(statusChanged)="handleChildNotification($event)"`.

### Expected Output
* A parent box containing a nested dashed child box.
* The child displays: "Name: Sahil Gedam", "Role: Frontend Architect", and a bullet list of hobbies.
* Clicking "Notify Parent" renders a blue notification banner: "Message from Child: Child action completed successfully!" at the top of the parent box.

### Common Interview Follow-up Questions
1. **What is EventEmitter?**
   * An Angular class extending RxJS `Subject` used to emit custom events.
2. **Why use `@Input` and `@Output` instead of a shared service for parent-child communication?**
   * They keep components highly reusable, self-contained, and clean for direct parent-child relationships.
3. **Can we pass dynamic objects/arrays via `@Input`? How does Angular track changes?**
   * Yes. However, Angular's default change detection checks object reference changes, not internal mutations. Mutating property values inside an object won't trigger change detection unless you copy the object (`{...obj}`).

### Alternative Solution (Angular 16+ Signals API)
```typescript
// Child:
import { input, output } from '@angular/core';
export class ChildComponent {
  user = input.required<{ name: string; role: string }>();
  hobbies = input<string[]>([]);
  statusChanged = output<string>();

  sendNotification() {
    this.statusChanged.emit('Child action completed successfully!');
  }
}
```

### Common Mistakes
* Forgetting to import the Child component in the Parent's `imports: [...]` array when using Standalone components.
* Modifying input property values directly inside the child component (violates unidirectional data flow).

### Best Practices
* Always type `@Input` properties explicitly. Avoid using `any`.
* Use the readonly prefix or modern Signal-based inputs (`input()`) for stricter type-safety.

---

## SECTION 2: Data Binding

### Question
> Create a single component showcasing Interpolation, Property Binding, Event Binding, and Two-way binding using both standard `ngModel` and manual Event binding.

### Difficulty
Easy

### Concepts Tested
* Interpolation (`{{ }}`)
* Property Binding (`[property]`)
* Event Binding (`(event)`)
* Two-way Data Binding (`[(ngModel)]`)

### Folder Structure
```text
src/
  app/
    components/
      binding-demo/
        binding-demo.component.ts
        binding-demo.component.html
```

### Complete Code

#### `src/app/components/binding-demo/binding-demo.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-binding-demo',
  standalone: true,
  imports: [CommonModule, FormsModule],
  templateUrl: './binding-demo.component.html',
  styleUrls: []
})
export class BindingDemoComponent {
  title = 'Deloitte Interview Practice';
  isDisabled = true;
  userInput = 'Initial text';
  manualInput = '';

  toggleDisable() {
    this.isDisabled = !this.isDisabled;
  }

  handleManualInput(event: Event) {
    const target = event.target as HTMLInputElement;
    this.manualInput = target.value;
  }
}
```

#### `src/app/components/binding-demo/binding-demo.component.html`
```html
<div style="max-width: 450px; margin: 20px auto; padding: 20px; border: 1px solid #ddd; border-radius: 8px; font-family: Arial, sans-serif;">
  <h3>Data Binding Demo</h3>
  
  <!-- Interpolation -->
  <p><strong>Topic:</strong> {{ title }}</p>

  <!-- Property Binding -->
  <button [disabled]="isDisabled" style="margin-right: 10px; padding: 6px 12px;">Target Button</button>
  
  <!-- Event Binding -->
  <button (click)="toggleDisable()" style="padding: 6px 12px; cursor: pointer;">Toggle Disable State</button>

  <!-- Two-Way Binding with ngModel -->
  <div style="margin-top: 15px;">
    <label>Using ngModel:</label>
    <input [(ngModel)]="userInput" style="width: 100%; padding: 8px; margin-top: 5px; box-sizing: border-box;" />
    <p>Value: <em>{{ userInput }}</em></p>
  </div>

  <!-- Manual Event Binding (Equivalent to [(ngModel)]) -->
  <div style="margin-top: 15px;">
    <label>Manual Input Binding:</label>
    <input [value]="manualInput" (input)="handleManualInput($event)" style="width: 100%; padding: 8px; margin-top: 5px; box-sizing: border-box;" />
    <p>Manual Value: <em>{{ manualInput }}</em></p>
  </div>
</div>
```

### Step-by-step Explanation
1. **Interpolation**: Renders typescript variables inside HTML using double curly braces.
2. **Property Binding**: `[disabled]="isDisabled"` maps the component's boolean state to the button's DOM property.
3. **Event Binding**: `(click)="toggleDisable()"` listens to user actions and calls the handler.
4. **`[(ngModel)]`**: Requires `FormsModule`. It keeps input box value synchronized with `userInput` bidirectional.
5. **Manual alternative**: `[value]="manualInput" (input)="..."` achieves the exact same thing manually by writing to the value property and listening to inputs.

### Expected Output
* Display text: "Topic: Deloitte Interview Practice".
* A disabled button next to a button that toggles its state on click.
* Two text input fields; typing inside updates their respective paragraphs immediately.

### Common Interview Follow-up Questions
1. **What is the difference between attributes and properties?**
   * Attributes are defined by HTML, while properties are defined by the DOM. Angular binds to DOM properties.
2. **Which module must be imported to use `ngModel`?**
   * `FormsModule` from `@angular/forms`.
3. **How does `[(ngModel)]` work behind the scenes?**
   * It is syntactic sugar combining Property binding `[ngModel]` and Event binding `(ngModelChange)`.

### Alternative Solution
Using Signals for properties:
```typescript
import { signal } from '@angular/core';
export class BindingDemoComponent {
  userInput = signal('Initial text');
}
// HTML: <input [ngModel]="userInput()" (ngModelChange)="userInput.set($event)" />
```

### Common Mistakes
* Forgetting to import `FormsModule` in standalone components, resulting in `Can't bind to 'ngModel' since it isn't a known property of 'input'`.

### Best Practices
* Prefer property binding over string interpolation when binding to non-string DOM attributes.

---

## SECTION 3: Structural Directives

### Question
> Implement an items list using `*ngFor` with a trackBy function, render conditional warning banners using `*ngIf` / `else` templates, and build a view selector using `*ngSwitch`. Show how nested `*ngFor` lists are handled.

### Difficulty
Medium

### Concepts Tested
* Structural directives (`*ngIf`, `*ngFor`, `*ngSwitch`)
* Performance optimization using `trackBy`
* Nested templates (`ng-template`)

### Folder Structure
```text
src/
  app/
    components/
      directives-demo/
        directives-demo.component.ts
        directives-demo.component.html
```

### Complete Code

#### `src/app/components/directives-demo/directives-demo.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

interface Project {
  id: number;
  name: string;
  tasks: string[];
}

@Component({
  selector: 'app-directives-demo',
  standalone: true,
  imports: [CommonModule],
  templateUrl: './directives-demo.component.html',
  styleUrls: []
})
export class DirectivesDemoComponent {
  isLoggedIn = true;
  viewMode = 'list'; // can be 'list', 'grid', or 'compact'

  projects: Project[] = [
    { id: 101, name: 'Deloitte Portal', tasks: ['Design UI', 'Setup Auth'] },
    { id: 102, name: 'Core API Integration', tasks: ['Express Setup', 'Unit Testing'] }
  ];

  trackByProjectId(index: number, item: Project): number {
    return item.id;
  }

  toggleLogin() {
    this.isLoggedIn = !this.isLoggedIn;
  }

  changeView(mode: string) {
    this.viewMode = mode;
  }
}
```

#### `src/app/components/directives-demo/directives-demo.component.html`
```html
<div style="max-width: 500px; margin: 20px auto; padding: 20px; border: 1px solid #bbb; border-radius: 8px; font-family: sans-serif;">
  <h3>Structural Directives Demo</h3>

  <!-- *ngIf / else -->
  <div *ngIf="isLoggedIn; else loginPrompt" style="margin-bottom: 20px;">
    <span style="color: green;">✔ Welcome back!</span>
    <button (click)="toggleLogin()" style="margin-left: 10px; padding: 4px 8px;">Logout</button>
  </div>
  <ng-template #loginPrompt>
    <div style="margin-bottom: 20px;">
      <span style="color: red;">✘ Please log in to view details.</span>
      <button (click)="toggleLogin()" style="margin-left: 10px; padding: 4px 8px;">Login</button>
    </div>
  </ng-template>

  <!-- Conditional Container content -->
  <div *ngIf="isLoggedIn">
    <!-- *ngSwitch -->
    <div style="margin-bottom: 15px;">
      <button (click)="changeView('list')">List View</button>
      <button (click)="changeView('grid')" style="margin-left:5px;">Grid View</button>
      <button (click)="changeView('compact')" style="margin-left:5px;">Compact</button>
    </div>

    <div [ngSwitch]="viewMode" style="border: 1px dashed #777; padding: 10px; margin-bottom: 15px; border-radius: 4px;">
      <div *ngSwitchCase="'list'"><strong>Displaying as Standard List:</strong></div>
      <div *ngSwitchCase="'grid'"><strong>Displaying as Modern Grid Cards:</strong></div>
      <div *ngSwitchDefault><strong>Displaying Simple Compact Text:</strong></div>
    </div>

    <!-- Nested *ngFor & trackBy -->
    <h4>Projects & Tasks:</h4>
    <div *ngFor="let proj of projects; trackBy: trackByProjectId" style="border-bottom: 1px solid #eee; padding: 8px 0;">
      <h5>Project: {{ proj.name }} (ID: {{ proj.id }})</h5>
      <ul>
        <li *ngFor="let task of proj.tasks">{{ task }}</li>
      </ul>
    </div>
  </div>
</div>
```

### Step-by-step Explanation
1. **`*ngIf="isLoggedIn; else loginPrompt"`**: Checks state. If truthy, renders element; otherwise, renders reference template `#loginPrompt`.
2. **`[ngSwitch]="viewMode"`**: Dynamically matches value against cases `*ngSwitchCase="'list'"` and renders the matched node.
3. **`trackBy`**: Returns a unique identifier (`proj.id`). This prevents Angular from destroying and recreating the entire DOM collection whenever items change.

### Expected Output
* Displaying "Welcome back!" and logout option.
* Buttons to change view states matching switch template updates.
* Detailed list of project items with nested bullet points. Clicking logout hides the list and displays the red login warning.

### Common Interview Follow-up Questions
1. **Why is `trackBy` important in `*ngFor` loops?**
   * It boosts rendering performance. By default, Angular uses object identity to trace changes. If arrays are re-fetched from a server, all DOM elements are recreated. `trackBy` tracks unique IDs, updating only modified nodes.
2. **Can you apply multiple structural directives on a single HTML host element?**
   * No. A tag can host at most one structural directive. Use `<ng-container>` to apply multiple layout directives without injecting extra wrapper DOM elements.
3. **What is the difference between `<ng-template>` and `<ng-container>`?**
   * `<ng-template>` represents a template definition that is not rendered until requested. `<ng-container>` is an structural element that groups DOM elements but itself does not produce a tag in the final layout.

### Alternative Solution (Angular 17+ Control Flow Syntax)
Angular 17 introduced built-in control flow which eliminates imports of `CommonModule` directives:
```html
@if (isLoggedIn) {
  <p>Welcome back!</p>
} @else {
  <p>Please log in.</p>
}

@for (proj of projects; track proj.id) {
  <p>{{ proj.name }}</p>
}
```

### Common Mistakes
* Writing `trackBy` in HTML as `trackBy: trackByProjectId()` (executing the function immediately) rather than assigning the reference `trackBy: trackByProjectId`.
* Putting `*` in front of `ngSwitch` (e.g., `*ngSwitch="..."`). The switch parent uses property binding syntax `[ngSwitch]`, and children use structural `*ngSwitchCase`.

---

## SECTION 4: Attribute Directives

### Question
> Create a custom directive named `appHighlight` that highlights elements yellow on hover, disables clicks if a configuration flag is enabled, and dynamically changes the element's opacity.

### Difficulty
Medium

### Concepts Tested
* Custom Attribute Directives
* `@Directive` decorator
* `ElementRef` and `Renderer2`
* `@HostListener` and `@HostBinding`

### Folder Structure
```text
src/
  app/
    directives/
      highlight.directive.ts
    components/
      directive-test/
        directive-test.component.ts
```

### Complete Code

#### `src/app/directives/highlight.directive.ts`
```typescript
import { Directive, ElementRef, Renderer2, HostListener, Input, HostBinding } from '@angular/core';

@Directive({
  selector: '[appHighlight]',
  standalone: true
})
export class HighlightDirective {
  @Input() appHighlightColor = 'yellow';
  @Input() disableInteraction = false;

  @HostBinding('style.opacity') opacity = '1.0';

  constructor(private el: ElementRef, private renderer: Renderer2) {}

  @HostListener('mouseenter') onMouseEnter() {
    if (this.disableInteraction) return;
    this.highlight(this.appHighlightColor);
    this.opacity = '0.7';
  }

  @HostListener('mouseleave') onMouseLeave() {
    this.highlight(null);
    this.opacity = '1.0';
  }

  @HostListener('click', ['$event']) onClick(event: Event) {
    if (this.disableInteraction) {
      event.preventDefault();
      event.stopPropagation();
      alert('Clicks are disabled on this element!');
    }
  }

  private highlight(color: string | null) {
    this.renderer.setStyle(this.el.nativeElement, 'backgroundColor', color);
  }
}
```

#### `src/app/components/directive-test/directive-test.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HighlightDirective } from '../../directives/highlight.directive';

@Component({
  selector: 'app-directive-test',
  standalone: true,
  imports: [CommonModule, HighlightDirective],
  template: `
    <div style="padding: 20px; font-family: Arial, sans-serif; max-width: 400px; margin: auto;">
      <h4>Custom Directive Demo</h4>
      <p appHighlight appHighlightColor="#e6f7ff" style="padding: 10px; border: 1px solid #ccc; cursor: pointer;">
        Hover over me (Highlights Light Blue)
      </p>
      
      <p [appHighlight]="'#ffe58f'" [disableInteraction]="true" style="padding: 10px; border: 1px solid #ccc; cursor: not-allowed; margin-top: 10px;">
        Hover disabled / Click triggers Alert
      </p>
    </div>
  `
})
export class DirectiveTestComponent {}
```

### Step-by-step Explanation
1. **`@Directive`**: Marks the class. The CSS selector `[appHighlight]` target elements as attributes.
2. **`ElementRef` & `Renderer2`**: Standard Angular services to safely manipulate direct host DOM attributes without breaking platform independence (e.g. server-side rendering support).
3. **`@HostListener`**: Hooks custom DOM events on the host element (`mouseenter`, `mouseleave`, `click`).
4. **`@HostBinding`**: Dynamically binds CSS property values (`opacity`) on the host element.

### Expected Output
* Text box 1 highlights light blue and dims to `0.7` opacity on hover.
* Text box 2 triggers a dialog box: "Clicks are disabled on this element!" when clicked.

### Common Interview Follow-up Questions
1. **Why should we use `Renderer2` instead of direct native DOM manipulations (`element.style.color = 'red'`)?**
   * Using Renderer2 ensures that the application is compatible with Web Workers, Server-Side Rendering (SSR/Universal), and Testing environments where the native `window` or `document` objects are unavailable.
2. **What does `@HostBinding` do?**
   * It allows us to set properties (like CSS styles or DOM classes) of the element hosting the directive directly from the typescript class.
3. **How do custom directives handle component Inputs?**
   * Directives can define standard `@Input()` properties just like components. They are evaluated on the host tag.

### Alternative Solution (Direct Binding)
If only style updates are needed, inline directives can also be written via standard attribute bindings:
```html
<p [style.background-color]="hovered ? 'yellow' : 'transparent'" (mouseenter)="hovered=true" (mouseleave)="hovered=false"></p>
```

### Common Mistakes
* Forgetting to set `standalone: true` on the custom directive and import it into the testing component's `imports` array.

---

## SECTION 5: Pipes

### Question
> Build a custom pure pipe named `filterItems` that filters an array of objects based on a property name and search query. Explain the difference between pure and impure pipes.

### Difficulty
Medium

### Concepts Tested
* Angular Pipes (`@Pipe` decorator)
* PipeTransform Interface
* Pure vs Impure Pipe change detection behavior
* Built-in pipes usage

### Folder Structure
```text
src/
  app/
    pipes/
      filter-items.pipe.ts
    components/
      pipes-demo/
        pipes-demo.component.ts
```

### Complete Code

#### `src/app/pipes/filter-items.pipe.ts`
```typescript
import { Pipe, PipeTransform } from '@angular/core';

interface DemoItem {
  name: string;
  category: string;
}

@Pipe({
  name: 'filterItems',
  pure: true, // Default is true
  standalone: true
})
export class FilterItemsPipe implements PipeTransform {
  transform(items: DemoItem[], searchTerm: string, property: keyof DemoItem): DemoItem[] {
    if (!items || !searchTerm) {
      return items;
    }
    return items.filter(item => 
      item[property].toLowerCase().includes(searchTerm.toLowerCase())
    );
  }
}
```

#### `src/app/components/pipes-demo/pipes-demo.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FilterItemsPipe } from '../../pipes/filter-items.pipe';
import { FormsModule } from '@angular/forms';

@Component({
  selector: 'app-pipes-demo',
  standalone: true,
  imports: [CommonModule, FormsModule, FilterItemsPipe],
  template: `
    <div style="max-width: 400px; margin: 20px auto; padding: 20px; border: 1px solid #ccc; border-radius: 8px; font-family: sans-serif;">
      <h4>Pipes & Filtering Demo</h4>
      
      <!-- Built-in Pipes -->
      <p><strong>Date Pipe:</strong> {{ today | date:'mediumDate' }}</p>
      <p><strong>Currency:</strong> {{ salary | currency:'INR':'symbol' }}</p>
      <p><strong>Uppercase:</strong> {{ 'deloitte' | uppercase }}</p>

      <hr />
      
      <!-- Custom Pure Filter Pipe -->
      <label>Search Products:</label>
      <input [(ngModel)]="searchQuery" placeholder="Filter by name..." style="width: 100%; padding: 8px; margin: 8px 0;" />
      
      <ul>
        <li *ngFor="let product of (products | filterItems:searchQuery:'name')">
          {{ product.name }} ({{ product.category }})
        </li>
      </ul>
    </div>
  `
})
export class PipesDemoComponent {
  today = new Date();
  salary = 120000;
  searchQuery = '';

  products = [
    { name: 'MacBook Pro', category: 'Laptop' },
    { name: 'iPhone 15', category: 'Phone' },
    { name: 'iPad Air', category: 'Tablet' }
  ];
}
```

### Step-by-step Explanation
1. **`@Pipe` / `transform`**: Custom pipes implement the `PipeTransform` interface and define a `transform(value, ...args)` method which accepts values and formats them.
2. **Pure Pipe**: Marked by `pure: true`. It executes only when the input reference changes (e.g. primitive updates, object reference swaps).
3. **Built-in Pipes**: Displays formatting for common data types (`date`, `currency`, `uppercase`).

### Expected Output
* Rendered date: "Jul 12, 2026" (or equivalent current local date).
* Currency displays as `₹1,20,000.00`.
* List of products filters dynamically as the user types characters in the search text input.

### Common Interview Follow-up Questions
1. **What is the difference between a Pure Pipe and an Impure Pipe?**
   * A **Pure Pipe** runs only when input arguments change reference. An **Impure Pipe** runs on every single change detection cycle (highly resource intensive; avoid for computationally heavy filter loops).
2. **Why is it discouraged to use a custom Pipe for filtering arrays in production?**
   * If the pipe is impure, it recalculates on every mouse move/keyup, killing UI performance. If it's pure, updating items inside the array without mutating the array reference won't trigger re-filtering.
3. **What is the default value of the `pure` property in `@Pipe`?**
   * `true`.

### Alternative Solution (Filtering inside Component Controller)
Instead of pipes, handle filtering via component methods or reactive RxJS Streams:
```typescript
get filteredProducts() {
  return this.products.filter(p => p.name.toLowerCase().includes(this.searchQuery.toLowerCase()));
}
```

---

## SECTION 6: Forms (Reactive & Validation)

### Question
> Implement an Angular Reactive Form containing dynamic validations. Fields required: email, password, and optional phone. Password must contain at least 1 digit, 1 uppercase, 1 special character and be minimum 8 characters. Apply matching dynamic custom validators to check password match. Render descriptive validations.

### Difficulty
Hard / Medium

### Concepts Tested
* `FormBuilder`, `FormGroup`, `FormControl`
* Custom Validation Functions
* Validation Matcher
* `Validators.pattern` and conditional validators

### Folder Structure
```text
src/
  app/
    components/
      register-form/
        register-form.component.ts
        register-form.component.html
```

### Complete Code

#### `src/app/components/register-form/register-form.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule, AbstractControl, ValidationErrors } from '@angular/forms';

@Component({
  selector: 'app-register-form',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  templateUrl: './register-form.component.html',
  styleUrls: []
})
export class RegisterFormComponent {
  registerForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.registerForm = this.fb.group({
      email: ['', [Validators.required, Validators.email]],
      password: ['', [
        Validators.required, 
        Validators.minLength(8),
        Validators.pattern(/^(?=.*[0-9])(?=.*[A-Z])(?=.*[!@#$%^&*])/)
      ]],
      confirmPassword: ['', [Validators.required]]
    }, { validators: this.passwordMatchValidator });
  }

  // Custom Validator to check matching passwords
  passwordMatchValidator(control: AbstractControl): ValidationErrors | null {
    const password = control.get('password');
    const confirmPassword = control.get('confirmPassword');

    if (password && confirmPassword && password.value !== confirmPassword.value) {
      confirmPassword.setErrors({ passwordsMismatch: true });
      return { passwordsMismatch: true };
    }
    return null;
  }

  onSubmit() {
    if (this.registerForm.valid) {
      console.log('Form Submitted!', this.registerForm.value);
      alert('Registration Successful!');
      this.registerForm.reset();
    } else {
      this.registerForm.markAllAsTouched();
    }
  }
}
```

#### `src/app/components/register-form/register-form.component.html`
```html
<div style="max-width: 400px; margin: 30px auto; padding: 25px; border: 1px solid #d9d9d9; border-radius: 8px; font-family: sans-serif; box-shadow: 0 4px 12px rgba(0,0,0,0.15);">
  <h3 style="margin-top:0;">Register Form</h3>
  
  <form [formGroup]="registerForm" (ngSubmit)="onSubmit()">
    <!-- Email Field -->
    <div style="margin-bottom: 15px;">
      <label style="display:block; margin-bottom:5px;">Email:</label>
      <input type="email" formControlName="email" style="width:100%; padding:8px; border:1px solid #ccc; border-radius:4px;" />
      <div *ngIf="registerForm.get('email')?.touched && registerForm.get('email')?.errors" style="color:red; font-size:12px; margin-top:5px;">
        <span *ngIf="registerForm.get('email')?.hasError('required')">Email is required.</span>
        <span *ngIf="registerForm.get('email')?.hasError('email')">Invalid email address.</span>
      </div>
    </div>

    <!-- Password Field -->
    <div style="margin-bottom: 15px;">
      <label style="display:block; margin-bottom:5px;">Password:</label>
      <input type="password" formControlName="password" style="width:100%; padding:8px; border:1px solid #ccc; border-radius:4px;" />
      <div *ngIf="registerForm.get('password')?.touched && registerForm.get('password')?.errors" style="color:red; font-size:12px; margin-top:5px;">
        <span *ngIf="registerForm.get('password')?.hasError('required')">Password is required.</span>
        <span *ngIf="registerForm.get('password')?.hasError('minlength')">Must be at least 8 characters.</span>
        <span *ngIf="registerForm.get('password')?.hasError('pattern')">Must contain at least 1 digit, 1 uppercase character, and 1 special symbol.</span>
      </div>
    </div>

    <!-- Confirm Password Field -->
    <div style="margin-bottom: 15px;">
      <label style="display:block; margin-bottom:5px;">Confirm Password:</label>
      <input type="password" formControlName="confirmPassword" style="width:100%; padding:8px; border:1px solid #ccc; border-radius:4px;" />
      <div *ngIf="registerForm.get('confirmPassword')?.touched && registerForm.get('confirmPassword')?.errors" style="color:red; font-size:12px; margin-top:5px;">
        <span *ngIf="registerForm.get('confirmPassword')?.hasError('required')">Confirm Password is required.</span>
        <span *ngIf="registerForm.get('confirmPassword')?.hasError('passwordsMismatch')">Passwords do not match.</span>
      </div>
    </div>

    <button type="submit" [disabled]="registerForm.invalid" style="width:100%; background-color:#2f54eb; color:white; font-weight:bold; border:none; padding:10px; border-radius:4px; cursor:pointer;">
      Register
    </button>
  </form>
</div>
```

### Step-by-step Explanation
1. **`FormBuilder`**: Injectable service providing syntactical helpers to build complex form trees.
2. **`Validators`**: Combined built-in checkers (`required`, `email`, `minLength`, `pattern` regex).
3. **Cross-Field Custom Validator**: `passwordMatchValidator` runs validation at the group container layer, dynamically setting errors to fields depending on mismatch state.

### Expected Output
* Form disabled by default.
* Interactive red feedback labels indicating missing fields or password strength checks on user blur/focus updates.
* Successful validation matching unlocks the registration button. Clicking it displays a dialog stating `Registration Successful!`.

### Common Interview Follow-up Questions
1. **What is the difference between Reactive Forms and Template-Driven Forms?**
   * **Reactive Forms**: Form structure built in TypeScript class, synchronous validation, predictable, immutability of state, and highly testable.
   * **Template-Driven Forms**: Structure built in HTML using `ngModel`, asynchronous, relies heavily on template directives, hard to write unit tests for.
2. **How do you dynamically add validation rules after form initialization?**
   * Use `formControl.setValidators([Validators.required])` followed by calling `formControl.updateValueAndValidity()`.
3. **What is `AbstractControl`?**
   * The base class for FormControls, FormGroups, and FormArrays in Angular.

### Alternative Solution (Template Driven Alternative)
Not recommended for complex validators due to excessive HTML binding complexity. Custom directives must be declared implementing the `NG_VALIDATORS` token.

---

## SECTION 7: Angular Services & DI

### Question
> Create a Singleton service that stores a shared counter value. Expose this value as a `BehaviorSubject` observable and show how two separate sibling components can increment/decrement this value.

### Difficulty
Medium

### Concepts Tested
* Service Creation (`@Injectable`)
* Dependency Injection
* Sharing state using RxJS `BehaviorSubject`
* Observable subscriptions

### Folder Structure
```text
src/
  app/
    services/
      counter.service.ts
    components/
      sibling-one/
        sibling-one.component.ts
      sibling-two/
        sibling-two.component.ts
```

### Complete Code

#### `src/app/services/counter.service.ts`
```typescript
import { Injectable } from '@angular/core';
import { BehaviorSubject, Observable } from 'rxjs';

@Injectable({
  providedIn: 'root' // Declares the service as singleton at application root level
})
export class CounterService {
  private counterSubject = new BehaviorSubject<number>(0);
  
  // Expose as Readonly Observable to prevent external mutations
  counter$: Observable<number> = this.counterSubject.asObservable();

  increment() {
    this.counterSubject.next(this.counterSubject.value + 1);
  }

  decrement() {
    this.counterSubject.next(this.counterSubject.value - 1);
  }
}
```

#### `src/app/components/sibling-one/sibling-one.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { CounterService } from '../../services/counter.service';

@Component({
  selector: 'app-sibling-one',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div style="border:1px solid #adc6ff; padding:15px; border-radius:6px; margin: 10px;">
      <h5>Sibling One Component</h5>
      <p>Current Count: <strong>{{ currentCount }}</strong></p>
      <button (click)="increment()" style="background-color:#1d39c4; color:white; padding:6px 12px; border:none; cursor:pointer;">Increment</button>
    </div>
  `
})
export class SiblingOneComponent {
  currentCount = 0;

  constructor(private counterService: CounterService) {
    this.counterService.counter$.subscribe(val => {
      this.currentCount = val;
    });
  }

  increment() {
    this.counterService.increment();
  }
}
```

#### `src/app/components/sibling-two/sibling-two.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { CounterService } from '../../services/counter.service';

@Component({
  selector: 'app-sibling-two',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div style="border:1px solid #ffd666; padding:15px; border-radius:6px; margin: 10px;">
      <h5>Sibling Two Component</h5>
      <p>Current Count: <strong>{{ currentCount }}</strong></p>
      <button (click)="decrement()" style="background-color:#d48806; color:white; padding:6px 12px; border:none; cursor:pointer;">Decrement</button>
    </div>
  `
})
export class SiblingTwoComponent {
  currentCount = 0;

  constructor(private counterService: CounterService) {
    this.counterService.counter$.subscribe(val => {
      this.currentCount = val;
    });
  }

  decrement() {
    this.counterService.decrement();
  }
}
```

### Step-by-step Explanation
1. **`providedIn: 'root'`**: Registers service globally so Angular provides the same instantiation across the entire project scope.
2. **`BehaviorSubject`**: Stores state and emits value updates immediately to new subscribers upon subscription.
3. **`asObservable()`**: Exposes standard observable methods so clients cannot call `.next()` directly.

### Expected Output
* Two nested Sibling components inside the UI.
* Clicking "Increment" in Sibling One immediately reflects the updated state inside Sibling Two, and clicking "Decrement" in Sibling Two updates Sibling One.

### Common Interview Follow-up Questions
1. **What is a Singleton Service?**
   * A service class instantiated only once per application scope.
2. **What is the difference between `Subject` and `BehaviorSubject`?**
   * `Subject` does not hold a default initial value or emit current state upon subscription. `BehaviorSubject` requires a default state and emits the latest stored value to any new subscriber immediately.
3. **How do you make a service non-singleton?**
   * Provide it in the component's `providers: [...]` array instead of `providedIn: 'root'`.

---

## SECTION 8: HTTP & HTTP Client

### Question
> Implement an Employee Management Service that makes GET, POST, PUT, and DELETE API requests using Angular `HttpClient`. Pass headers, query parameters, handle errors gracefully using `catchError`, include timeout, and retry failing calls twice.

### Difficulty
Hard / Medium

### Concepts Tested
* Angular `HttpClient`
* HTTP Headers (`HttpHeaders`) and Query Parameters (`HttpParams`)
* RxJS Error Handling Operators (`catchError`, `retry`, `timeout`)

### Folder Structure
```text
src/
  app/
    services/
      employee.service.ts
    models/
      employee.model.ts
```

### Complete Code

#### `src/app/models/employee.model.ts`
```typescript
export interface Employee {
  id?: number;
  name: string;
  role: string;
}
```

#### `src/app/services/employee.service.ts`
```typescript
import { Injectable } from '@angular/core';
import { HttpClient, HttpHeaders, HttpParams, HttpErrorResponse } from '@angular/common/http';
import { Observable, throwError } from 'rxjs';
import { catchError, retry, timeout } from 'rxjs/operators';
import { Employee } from '../models/employee.model';

@Injectable({
  providedIn: 'root'
})
export class EmployeeService {
  private apiUrl = 'https://jsonplaceholder.typicode.com/users'; // Fake REST target

  constructor(private http: HttpClient) {}

  // GET with headers & params
  getEmployees(roleFilter?: string): Observable<Employee[]> {
    let params = new HttpParams();
    if (roleFilter) {
      params = params.set('role', roleFilter);
    }

    const headers = new HttpHeaders()
      .set('Authorization', 'Bearer dummy-token-123')
      .set('Content-Type', 'application/json');

    return this.http.get<Employee[]>(this.apiUrl, { headers, params }).pipe(
      timeout(5000), // Timeout after 5 seconds
      retry(2),      // Retry 2 times if request fails
      catchError(this.handleError)
    );
  }

  // POST (Create)
  createEmployee(employee: Employee): Observable<Employee> {
    return this.http.post<Employee>(this.apiUrl, employee).pipe(
      catchError(this.handleError)
    );
  }

  // PUT (Update)
  updateEmployee(id: number, employee: Employee): Observable<Employee> {
    return this.http.put<Employee>(`${this.apiUrl}/${id}`, employee).pipe(
      catchError(this.handleError)
    );
  }

  // DELETE
  deleteEmployee(id: number): Observable<any> {
    return this.http.delete<any>(`${this.apiUrl}/${id}`).pipe(
      catchError(this.handleError)
    );
  }

  private handleError(error: HttpErrorResponse) {
    let errorMessage = 'An unknown error occurred!';
    if (error.error instanceof ErrorEvent) {
      // Client-side / Network error
      errorMessage = `Client error: ${error.error.message}`;
    } else {
      // Server-side error
      errorMessage = `Server Error Code: ${error.status}\nMessage: ${error.message}`;
    }
    console.error(errorMessage);
    return throwError(() => new Error(errorMessage));
  }
}
```

### Step-by-step Explanation
1. **HttpParams & HttpHeaders**: Immutable helper objects to construct safe URL parameters and request metadata.
2. **`pipe()` operators**:
   * `timeout(5000)`: Rejects if the server does not respond within 5s.
   * `retry(2)`: Automates re-triggering of failed requests.
   * `catchError`: Intercepts client or server exceptions, formatting user-friendly errors before piping downstream.

### Expected Output
* Services return standard typescript observables. Testing client receives formatted array responses or descriptive JavaScript Errors inside `subscribe({error})` when endpoint paths are missing.

### Common Interview Follow-up Questions
1. **What is the return type of HttpClient methods?**
   * Cold Observables emitting a single response object before completing.
2. **Why do we call `.pipe()` inside our HTTP services?**
   * To chain RxJS operators for handling failures, timeouts, formatting, and retries.
3. **What is `HttpBackend` and when do we use it?**
   * An alternative HTTP handler that bypasses the configured HTTP interceptor chain. Useful when making external API calls where interceptor tokens are not required.

---

## SECTION 9: RxJS Operators

### Question
> Build a live search filter component using a search input text box. Utilize `debounceTime`, `distinctUntilChanged`, `switchMap`, and `catchError` operators to query a remote endpoint asynchronously. Avoid memory leaks by cleaning subscriptions using `takeUntil`.

### Difficulty
Hard

### Concepts Tested
* RxJS Subjects (`Subject`)
* Stream Transformation Operators (`switchMap`, `debounceTime`, `distinctUntilChanged`)
* Memory Leak prevention (`Subject` + `takeUntil`)

### Folder Structure
```text
src/
  app/
    components/
      live-search/
        live-search.component.ts
```

### Complete Code

#### `src/app/components/live-search/live-search.component.ts`
```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import { CommonModule } from '@angular/common';
import { HttpClient } from '@angular/common/http';
import { Subject } from 'rxjs';
import { debounceTime, distinctUntilChanged, switchMap, takeUntil, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

@Component({
  selector: 'app-live-search',
  standalone: true,
  imports: [CommonModule],
  template: `
    <div style="max-width: 400px; margin: 20px auto; padding: 20px; border: 1px solid #ccc; border-radius: 8px; font-family: sans-serif;">
      <h4>RxJS Live Search</h4>
      <input (input)="onKeyUp($event)" placeholder="Search users (e.g. Leanne)..." style="width:100%; padding:8px; box-sizing:border-box; margin-bottom:10px;" />
      
      <div *ngIf="loading">Searching...</div>
      
      <ul>
        <li *ngFor="let result of searchResults">{{ result.name }}</li>
      </ul>
    </div>
  `
})
export class LiveSearchComponent implements OnInit, OnDestroy {
  private searchSubject = new Subject<string>();
  private destroy$ = new Subject<void>();

  searchResults: any[] = [];
  loading = false;

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.searchSubject.pipe(
      debounceTime(400),                 // Wait for 400ms pause
      distinctUntilChanged(),            // Only trigger if value changes
      switchMap(query => {
        if (!query.trim()) {
          return of([]);                 // Return empty array observable instantly
        }
        this.loading = true;
        return this.http.get<any[]>(`https://jsonplaceholder.typicode.com/users?q=${query}`).pipe(
          catchError(() => {
            this.loading = false;
            return of([]);
          })
        );
      }),
      takeUntil(this.destroy$)           // Clean subscription upon component destruction
    ).subscribe(data => {
      this.searchResults = data;
      this.loading = false;
    });
  }

  onKeyUp(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    this.searchSubject.next(value);
  }

  ngOnDestroy() {
    this.destroy$.next();
    this.destroy$.complete();
  }
}
```

### Step-by-step Explanation
1. **`Subject`**: Emits keyup values onto a reactive RxJS stream pipeline.
2. **`debounceTime(400)`**: Prevents sending API requests for every single keystroke. Wait for the user to pause typing.
3. **`distinctUntilChanged()`**: Prevents matching identical successive text selections (e.g. backspacing and typing back same key).
4. **`switchMap()`**: Cancels previous unresolved requests if a new key value starts a new API request.
5. **`takeUntil(destroy$)`**: Closes the open observable when `ngOnDestroy` fires to prevent memory leaks.

### Expected Output
* Typing "Le" shows "Searching...". If you type fast, only 1 request is fired. Displays filtered user names matching queries.

### Common Interview Follow-up Questions
1. **What is the difference between `switchMap`, `mergeMap`, `concatMap`, and `exhaustMap`?**
   * `switchMap`: Cancels current active inner observable when a new value arrives.
   * `mergeMap`: Runs all inner observables in parallel.
   * `concatMap`: Queues inner observables, executing them sequentially one after another.
   * `exhaustMap`: Ignores new incoming values while current active inner observable is executing.
2. **Why is `AsyncPipe` preferred over manual subscriptions?**
   * It handles subscriptions and automatic unsubscriptions inside HTML templates automatically, eliminating manual `takeUntil` setups.

---

## SECTION 10: Routing

### Question
> Implement basic routing configurations, child routes, custom path params parser, and a Route Guard preventing access to a dashboard route if authorization tokens are missing.

### Difficulty
Medium

### Concepts Tested
* Angular Router config
* `CanActivateFn` (Functional Route Guard)
* Route params extraction (`ActivatedRoute`)

### Folder Structure
```text
src/
  app/
    guards/
      auth.guard.ts
    components/
      dashboard/
        dashboard.component.ts
      login/
        login.component.ts
```

### Complete Code

#### `src/app/guards/auth.guard.ts`
```typescript
import { inject } from '@angular/core';
import { CanActivateFn, Router } from '@angular/router';

export const authGuard: CanActivateFn = (route, state) => {
  const router = inject(Router);
  const token = localStorage.getItem('jwt');

  if (token) {
    return true; // Access granted
  }

  // Redirect to login if user is unauthenticated
  router.navigate(['/login']);
  return false;
};
```

#### `src/app/components/dashboard/dashboard.component.ts`
```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-dashboard',
  standalone: true,
  template: `
    <div style="padding:20px; font-family:sans-serif;">
      <h3>Dashboard (Protected Page)</h3>
      <p>Secure Content Visible!</p>
      <button (click)="logout()" style="padding: 6px 12px; background-color:red; color:white; border:none; cursor:pointer;">
        Logout
      </button>
    </div>
  `
})
export class DashboardComponent {
  constructor(private router: Router) {}

  logout() {
    localStorage.removeItem('jwt');
    this.router.navigate(['/login']);
  }
}
```

#### `src/app/components/login/login.component.ts`
```typescript
import { Component } from '@angular/core';
import { Router } from '@angular/router';

@Component({
  selector: 'app-login',
  standalone: true,
  template: `
    <div style="padding:20px; font-family:sans-serif; max-width:300px; margin:auto;">
      <h3>Login Component</h3>
      <button (click)="login()" style="padding: 10px; width:100%; background-color:green; color:white; border:none; cursor:pointer;">
        Mock Login (Sets JWT Token)
      </button>
    </div>
  `
})
export class LoginComponent {
  constructor(private router: Router) {}

  login() {
    localStorage.setItem('jwt', 'my-mock-token-999');
    this.router.navigate(['/dashboard']);
  }
}
```

#### `src/app/app.routes.ts`
```typescript
import { Routes } from '@angular/router';
import { LoginComponent } from './components/login/login.component';
import { authGuard } from './guards/auth.guard';

export const routes: Routes = [
  { path: 'login', component: LoginComponent },
  { 
    path: 'dashboard', 
    loadComponent: () => import('./components/dashboard/dashboard.component').then(m => m.DashboardComponent),
    canActivate: [authGuard]
  },
  { path: '', redirectTo: 'login', pathMatch: 'full' }
];
```

### Step-by-step Explanation
1. **Functional Guards**: Injecting modules directly within the Arrow function using the `inject(Router)` function context, bypassing class boilerplates.
2. **`loadComponent`**: Standard syntax for Lazy Loading component views.
3. **Route navigation**: Redirecting the viewport context to other screens using router actions.

### Expected Output
* Navigating to `/dashboard` directly forces redirection to `/login`.
* Clicking "Mock Login" stores a dummy token in localStorage, authorizing route access to `/dashboard`.

---

## SECTION 11: Lifecycle Hooks

### Question
> Create a component that implements `ngOnInit`, `ngOnChanges`, `ngAfterViewInit`, and `ngOnDestroy`. Log messages sequentially to explain when they fire and how to access input parameters.

### Difficulty
Medium

### Concepts Tested
* Angular Life-cycle interface triggers
* Component lifecycle sequence execution
* `SimpleChanges` values tracking

### Folder Structure
```text
src/
  app/
    components/
      lifecycle/
        lifecycle.component.ts
```

### Complete Code

```typescript
import { Component, OnInit, OnChanges, SimpleChanges, Input, AfterViewInit, OnDestroy } from '@angular/core';

@Component({
  selector: 'app-lifecycle',
  standalone: true,
  template: `<p>Lifecycle Tracker (Check logs) - Counter Value: {{ count }}</p>`
})
export class LifecycleComponent implements OnInit, OnChanges, AfterViewInit, OnDestroy {
  @Input() count = 0;

  constructor() {
    console.log('1. constructor: Component instance initialized. Input values not ready yet.');
  }

  ngOnChanges(changes: SimpleChanges) {
    console.log('2. ngOnChanges: Input values modified.', changes);
  }

  ngOnInit() {
    console.log('3. ngOnInit: Component is initialized. Inputs are available: ', this.count);
  }

  ngAfterViewInit() {
    console.log('4. ngAfterViewInit: DOM view content initialization resolved.');
  }

  ngOnDestroy() {
    console.log('5. ngOnDestroy: Cleanup phase before DOM ejection.');
  }
}
```

### Step-by-step Explanation
* **`ngOnChanges`**: Fires before `ngOnInit` and whenever input bindings change.
* **`ngOnInit`**: Runs once after inputs are bound.
* **`ngAfterViewInit`**: Triggers after the view (and child views) are rendered.
* **`ngOnDestroy`**: Clean up logic (e.g. unsubscribing observables, clearing timers) before component destruction.

---

## SECTION 12: Component Communication (Advanced)

### Question
> Implement component sibling communication using a template reference variable combined with `@ViewChild` to trigger a child method directly from a parent component.

### Difficulty
Medium

### Concepts Tested
* `@ViewChild` decorator properties
* Template Reference variables (`#ref`)

### Folder Structure
```text
src/
  app/
    components/
      adv-parent/
        adv-parent.component.ts
      adv-child/
        adv-child.component.ts
```

### Complete Code

#### `src/app/components/adv-child/adv-child.component.ts`
```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-adv-child',
  standalone: true,
  template: `<div style="padding:10px; background:#f5f5f5;">Child Status: {{ status }}</div>`
})
export class AdvChildComponent {
  status = 'Stopped';

  start() {
    this.status = 'Running...';
  }
}
```

#### `src/app/components/adv-parent/adv-parent.component.ts`
```typescript
import { Component, ViewChild } from '@angular/core';
import { AdvChildComponent } from '../adv-child/adv-child.component';

@Component({
  selector: 'app-adv-parent',
  standalone: true,
  imports: [AdvChildComponent],
  template: `
    <div style="border:1px solid #ddd; padding:20px; border-radius:8px;">
      <h4>Advanced ViewChild Parent</h4>
      <button (click)="triggerChildStart()" style="margin-bottom:10px;">Start Child from Parent</button>
      <app-adv-child #childRef></app-adv-child>
    </div>
  `
})
export class AdvParentComponent {
  @ViewChild('childRef') childInstance!: AdvChildComponent;

  triggerChildStart() {
    this.childInstance.start();
  }
}
```

### Step-by-step Explanation
1. **`#childRef`**: Declares a template reference variable pointing to the child component in the parent template.
2. **`@ViewChild('childRef')`**: Resolves the child reference inside the Parent typescript class, allowing access to the child instance's properties and methods.

---

## SECTION 13: Performance Optimization

### Question
> Demonstrate performance optimization using `ChangeDetectionStrategy.OnPush` along with a pure pipe to minimize change detection cycles.

### Difficulty
Medium

### Concepts Tested
* `ChangeDetectionStrategy.OnPush`
* Minimizing Change Detection cycles

### Folder Structure
```text
src/
  app/
    components/
      performance-demo/
        performance-demo.component.ts
```

### Complete Code

```typescript
import { Component, ChangeDetectionStrategy, Input } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-performance-demo',
  standalone: true,
  imports: [CommonModule],
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div style="border:1px solid #722ed1; padding:15px; border-radius:6px;">
      <h4>OnPush Performance Demo</h4>
      <p>Configured Input Value: {{ data.value }}</p>
    </div>
  `
})
export class PerformanceDemoComponent {
  @Input() data!: { value: string };
}
```

### Step-by-step Explanation
1. **`OnPush`**: Instructs Angular to run change detection only when the reference of `@Input()` objects change, or when an event originates from within the component itself.

---

## SECTION 14: Authentication & Storage

### Question
> Implement an Auth service that manages login state by storing JWT tokens, handles token checks, and provides cookie or localstorage helpers to logout components.

### Difficulty
Medium

### Concepts Tested
* Token storage (`localStorage` & Session storage options)
* State Clean up routines

### Complete Code

```typescript
import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class AuthService {
  storeToken(token: string) {
    localStorage.setItem('auth_token', token);
  }

  getToken(): string | null {
    return localStorage.getItem('auth_token');
  }

  isLoggedIn(): boolean {
    return !!this.getToken();
  }

  logout() {
    localStorage.removeItem('auth_token');
  }
}
```

---

## SECTION 15: Functional Interceptors

### Question
> Create a Functional Interceptor named `authInterceptor` to attach Bearer tokens to outgoing HTTP requests and intercept 401 error codes globally.

### Difficulty
Hard

### Concepts Tested
* Functional Interceptors
* `HttpInterceptorFn`
* Inject API usage

### Complete Code

#### `src/app/interceptors/auth.interceptor.ts`
```typescript
import { HttpInterceptorFn, HttpErrorResponse } from '@angular/common/http';
import { inject } from '@angular/core';
import { Router } from '@angular/router';
import { catchError } from 'rxjs/operators';
import { throwError } from 'rxjs';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const router = inject(Router);
  const token = localStorage.getItem('jwt');

  let modifiedReq = req;
  if (token) {
    modifiedReq = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }

  return next(modifiedReq).pipe(
    catchError((error: HttpErrorResponse) => {
      if (error.status === 401) {
        // Clear tokens and redirect
        localStorage.removeItem('jwt');
        router.navigate(['/login']);
      }
      return throwError(() => error);
    })
  );
};
```

---

## SECTION 16: Angular Material (Table & Paginator)

### Question
> Create a mock Angular Material table showcasing a simple UI displaying user info, featuring a paginator and basic sorting.

### Difficulty
Medium

### Concepts Tested
* Material Table modules (`MatTableModule`)
* Data Sources config (`MatTableDataSource`)

### Complete Code

```typescript
import { Component, ViewChild, OnInit } from '@angular/core';
import { MatTableModule, MatTableDataSource } from '@angular/material/table';
import { MatPaginator, MatPaginatorModule } from '@angular/material/paginator';
import { MatSort, MatSortModule } from '@angular/material/sort';

export interface UserItem {
  id: number;
  name: string;
  role: string;
}

@Component({
  selector: 'app-material-table',
  standalone: true,
  imports: [MatTableModule, MatPaginatorModule, MatSortModule],
  template: `
    <div class="mat-elevation-z8" style="max-width: 600px; margin: 20px auto;">
      <table mat-table [dataSource]="dataSource" matSort>
        <ng-container matColumnDef="id">
          <th mat-header-cell *matHeaderCellDef mat-sort-header> ID </th>
          <td mat-cell *matCellDef="let element"> {{element.id}} </td>
        </ng-container>

        <ng-container matColumnDef="name">
          <th mat-header-cell *matHeaderCellDef mat-sort-header> Name </th>
          <td mat-cell *matCellDef="let element"> {{element.name}} </td>
        </ng-container>

        <ng-container matColumnDef="role">
          <th mat-header-cell *matHeaderCellDef mat-sort-header> Role </th>
          <td mat-cell *matCellDef="let element"> {{element.role}} </td>
        </ng-container>

        <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
        <tr mat-row *matRowDef="let row; columns: displayedColumns;"></tr>
      </table>
      <mat-paginator [pageSizeOptions]="[5, 10]" showFirstLastButtons></mat-paginator>
    </div>
  `
})
export class MaterialTableComponent implements OnInit {
  displayedColumns: string[] = ['id', 'name', 'role'];
  dataSource = new MatTableDataSource<UserItem>([
    { id: 1, name: 'Alice', role: 'Architect' },
    { id: 2, name: 'Bob', role: 'Developer' },
    { id: 3, name: 'Charlie', role: 'QA' }
  ]);

  @ViewChild(MatPaginator) paginator!: MatPaginator;
  @ViewChild(MatSort) sort!: MatSort;

  ngOnInit() {
    this.dataSource.paginator = this.paginator;
    this.dataSource.sort = this.sort;
  }
}
```

---

## SECTION 17: Coding Mini Apps (To-Do List)

### Question
> Implement a complete Standalone To-Do application. Features needed: adding item, completing toggle item, deletion, and reactive search filter.

### Difficulty
Medium

### Concepts Tested
* Standalone form controls
* Array state operations
* Filtering bindings

### Complete Code

#### `src/app/components/todo/todo.component.ts`
```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms';

interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

@Component({
  selector: 'app-todo',
  standalone: true,
  imports: [CommonModule, FormsModule],
  template: `
    <div style="max-width: 400px; margin: 20px auto; padding: 20px; border: 1px solid #ccc; border-radius: 8px; font-family: sans-serif;">
      <h3>Todo List Mini-App</h3>
      
      <div style="display:flex; margin-bottom: 10px;">
        <input [(ngModel)]="newTodoText" placeholder="Add a new task..." style="flex:1; padding:8px; border:1px solid #ccc;" />
        <button (click)="addTodo()" style="background:#52c41a; color:white; border:none; padding:8px 12px; cursor:pointer;">Add</button>
      </div>

      <input [(ngModel)]="searchText" placeholder="Filter tasks..." style="width:100%; padding:8px; box-sizing:border-box; margin-bottom:15px; border:1px solid #ccc;" />

      <ul style="list-style:none; padding:0;">
        <li *ngFor="let todo of filteredTodos" style="display:flex; align-items:center; justify-content:space-between; padding:8px; border-bottom:1px solid #eee;">
          <span [style.text-decoration]="todo.completed ? 'line-through' : 'none'" (click)="toggleTodo(todo.id)" style="cursor:pointer; flex:1;">
            {{ todo.text }}
          </span>
          <button (click)="deleteTodo(todo.id)" style="background:#f5222d; color:white; border:none; padding:4px 8px; cursor:pointer;">Delete</button>
        </li>
      </ul>
    </div>
  `
})
export class TodoComponent {
  todos: Todo[] = [
    { id: 1, text: 'Review Angular Signals', completed: false },
    { id: 2, text: 'Prepare Express CRUD Mock APIs', completed: true }
  ];
  newTodoText = '';
  searchText = '';

  get filteredTodos() {
    return this.todos.filter(t => t.text.toLowerCase().includes(this.searchText.toLowerCase()));
  }

  addTodo() {
    if (!this.newTodoText.trim()) return;
    this.todos.push({
      id: Date.now(),
      text: this.newTodoText,
      completed: false
    });
    this.newTodoText = '';
  }

  toggleTodo(id: number) {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.completed = !todo.completed;
    }
  }

  deleteTodo(id: number) {
    this.todos = this.todos.filter(t => t.id !== id);
  }
}
```

---

## SECTION 18: JavaScript Coding for Angular Interviews

### Question
> Write JavaScript functions to: (1) Reverse a string, (2) Flatten a nested array, (3) Create a closure that functions as a Counter generator.

### Complete Code Examples

#### 1. Reverse String
```javascript
function reverseString(str) {
  return str.split('').reverse().join('');
}
```

#### 2. Flatten Array
```javascript
function flattenArray(arr) {
  return arr.reduce((acc, val) => 
    Array.isArray(val) ? acc.concat(flattenArray(val)) : acc.concat(val), 
  []);
}
```

#### 3. Closure Counter Example
```javascript
function createCounter() {
  let count = 0;
  return {
    increment() {
      count++;
      return count;
    },
    getValue() {
      return count;
    }
  };
}
```

---

## SECTION 19: Node + Express (Backend mock APIs)

### Question
> Build a simple Express Server that features standard CRUD endpoints (GET, POST, PUT, DELETE) managing an array of project items. Handle CORS and custom error middlewares.

### Complete Code

#### `server.js`
```javascript
const express = require('express');
const cors = require('cors');
const app = express();

app.use(cors());
app.use(express.json());

let projects = [
  { id: 1, name: 'Deloitte CMS' },
  { id: 2, name: 'Accenture Cloud Portal' }
];

// GET API
app.get('/api/projects', (req, res) => {
  res.json(projects);
});

// POST API
app.post('/api/projects', (req, res) => {
  const newProj = { id: Date.now(), name: req.body.name };
  projects.push(newProj);
  res.status(201).json(newProj);
});

// PUT API
app.put('/api/projects/:id', (req, res) => {
  const { id } = req.params;
  const project = projects.find(p => p.id === parseInt(id));
  if (!project) return res.status(404).json({ error: 'Project not found' });
  
  project.name = req.body.name;
  res.json(project);
});

// DELETE API
app.delete('/api/projects/:id', (req, res) => {
  const { id } = req.params;
  projects = projects.filter(p => p.id !== parseInt(id));
  res.json({ message: 'Project deleted successfully' });
});

// Global Error Handler Middleware
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).json({ error: 'Something went wrong inside the server' });
});

app.listen(3000, () => console.log('Mock server running on port 3000'));
```

---

## Top 25 Most Frequently Asked Angular Coding Questions

1. **Parent-Child Communication**: Pass data using `@Input` and `@Output` with `EventEmitter`. (Highest Probability)
2. **RxJS Live Search**: Query a backend API using `debounceTime`, `distinctUntilChanged`, and `switchMap`.
3. **Forms Validation**: Create a Reactive login/registration form with custom validators.
4. **Functional Route Guards**: Implement a guard that restricts access based on localStorage tokens.
5. **Functional Interceptor**: Attach authorization headers to outgoing HTTP requests.
6. **Pure Custom Pipe**: Create a filter pipe that searches an array of objects by property.
7. **Custom Directive**: Build a hover highlight directive.
8. **ViewChild**: Access a child component method or element from a parent component.
9. **Http Error Handling**: Implement error interception and retries in services.
10. **Lifecycle Hooks**: Log and explain the execution order of `OnChanges`, `OnInit`, `AfterViewInit`, and `OnDestroy`.
11. **BehaviorSubject**: Share state across unrelated components using a singleton service.
12. **OnPush Change Detection**: Optimize components with `ChangeDetectionStrategy.OnPush`.
13. **trackBy**: Implement performance-optimized item loops with `*ngFor` and trackBy.
14. **Template-Driven Forms**: Build a basic validated form using `ngModel`.
15. **HttpClient API Operations**: Write a service with standard REST actions (GET, POST, PUT, DELETE).
16. **Lazy Loading**: Route configuration using lazy-loaded Standalone components.
17. **Dynamic CSS Classes**: Toggle component classes dynamically using `ngClass`.
18. **Unsubscribing Streams**: Use `takeUntil` or `Subscription.unsubscribe` to prevent leaks.
19. **Content Projection**: Use `<ng-content>` to project markup templates dynamically.
20. **Dynamic Form Elements**: Create a dynamic `FormArray` for repeating list entries.
21. **Custom Structural Directive**: Implement a custom directive equivalent to `*ngIf`.
22. **Service Scopes**: Configure providers in components versus routing configurations.
23. **Async Pipe**: Bind observable data directly inside template HTML.
24. **HttpClient Params**: Build safe HTTP requests with query parameters.
25. **Signals**: Implement basic reactive data flows using Angular Signals (`signal`, `computed`, `effect`).
