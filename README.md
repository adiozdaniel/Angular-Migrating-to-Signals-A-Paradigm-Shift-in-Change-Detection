# Angular-Migrating-to-Signals-A-Paradigm-Shift-in-Change-Detection

[Read the article](https://dev.to/adiozdaniel/angular-migrating-to-signals-a-paradigm-shift-in-change-detection-3bmd)

Angular, one of the most popular web frameworks, has always enforced best practices, performance, developer experience and maintainability. With its migration to **Signals**, Angular is stepping into a new era of change detection and reactivity.

Let’s explore what Signals bring to the table, why Angular is making this shift and how it impacts development.  

---

### **What Are Signals?**  

At its core, a **Signal** represents a reactive state. Think of it as a way to encapsulate a value that notifies subscribers whenever it changes. Signals introduce a clean, predictable model for managing reactive state, moving away from Angular’s existing reliance on mutable state and complex change detection cycles.  

**In simple terms:**  
- A **Signal** holds a value.  
- When the value changes, it automatically propagates the change to all dependents.  

Here’s a basic example of Signals:  

```typescript  
import { signal } from '@angular/core';  

const count = signal(0);  

// Access the value  
console.log(count()); // Output: 0  

// Update the value  
count.set(1);  
console.log(count()); // Output: 1  

// Compute a derived value  
const doubled = computed(() => count() * 2);  
console.log(doubled()); // Output: 2  
```  

---

### **Why Migrate to Signals?**  

1. **Improved Performance**  
   Angular’s traditional change detection mechanism involves traversing the component tree to detect changes. This approach, while robust, can become a bottleneck in large applications. Signals optimize this by tracking dependencies and re-evaluating only when necessary.  

2. **Predictable Reactivity**  
   Signals introduce a clear data flow, making the application state predictable and easier to debug.  

3. **Fine-Grained Updates**  
   With Signals, Angular can minimize DOM updates by focusing only on parts of the UI affected by state changes, leading to better performance.  

4. **Developer Experience**  
   Signals simplify state management and align Angular more closely with modern reactive paradigms. Developers get a more intuitive and declarative API to work with.  

---

### **Key Features of Angular Signals**  

#### **Mutability with Control**  
Signals allow mutable state but enforce immutability-like behavior through controlled access and updates.  

```typescript  
const name = signal('Angular');  

// Update the signal  
name.set('Signals in Angular');  
console.log(name()); // Output: Signals in Angular  
```  

#### **Derived State**  
Derived signals allow you to compute values based on other signals. They automatically update when their dependencies change.  

```typescript  
const a = signal(2);  
const b = signal(3);  
const sum = computed(() => a() + b());  

console.log(sum()); // Output: 5  

a.set(5);  
console.log(sum()); // Output: 8  
```  

#### **Effectual Reactivity**  
Effects let you perform side effects in response to signal changes.  

```typescript  
const count = signal(0);  

// Re-actively log whenever the count signal changes
effect(() => {  
    console.log(`Count changed: ${count()}`);  
});  

count.set(1); // Logs: Count changed: 1  
```  

---

### **How Does This Impact Angular Developers?**  

#### **Adapting Existing Code**  
Migrating to Signals will require a shift in mindset and some updates to existing applications. However, Angular provides migration tools to ensure a smooth transition.  

#### **Simplified State Management**  
With Signals, Angular eliminates the need for additional state management libraries in many scenarios.  

#### **Easier Debugging**  
Since Signals track dependencies explicitly, understanding the flow of data and debugging issues becomes significantly easier.  

---

### **Getting Started with Signals in Angular**  

To start using Signals, ensure your Angular project is on the latest version (v19). The framework provides built-in utilities like `signal`, `computed`, and `effect` to manage reactive state.  

**Here’s an example of Signals in a component:**  

```typescript  
import { Component, signal, computed } from '@angular/core';  

@Component({  
    selector: 'app-counter',  // Defines the custom HTML tag <app-counter> for this component
    template: `  
        <div>  
            <p>Count: {{ count() }}</p>  <!-- Displays the current count -->
            <p>Double: {{ doubled() }}</p>  <!-- Displays the doubled value of count -->
            <button (click)="increment()">Increment</button>  <!-- Button to increment the count -->
        </div>  
    `  
})  
export class CounterComponent {  
    // A signal to hold the current count value, starting at 0
    count = signal(0);  

    // A computed signal that derives its value by doubling the current count
    doubled = computed(() => this.count() * 2);  

    // A method to increment the count signal by 1
    increment() {  
        this.count.set(this.count() + 1);  // Updates the count value
    }  
}   
```  

---

Here’s the updated migration example using a real-world component like the provided `SidenavComponent`. This includes separating the HTML and Typescript, and migrating it to use Signals.

---

### **Migrating to Signals: Real-World Example**  

#### **Before Migration**  

**Typescript (`sidenav.component.ts`)**  

```typescript
import { CommonModule } from '@angular/common'; // Provides common Angular directives like ngIf and ngFor
import { Component, HostListener, Input, Output, EventEmitter } from '@angular/core';

@Component({
  selector: 'app-sidenav', // Defines the custom HTML tag <app-sidenav> for this component
  imports: [CommonModule], // Declares dependencies used in the template
  templateUrl: './sidenav.component.html', // External HTML file for the component's structure
  styleUrls: ['./sidenav.component.scss'] // External CSS/SCSS file for styling the component
})
export class SidenavComponent {
  // Input property to initialize the sidenav's open state
  @Input() initialOpen = false;

  // Output property to emit events when the sidenav toggles
  @Output() sidenavToggled = new EventEmitter<boolean>();

  // Tracks whether the sidenav is open
  isOpen = this.initialOpen;

  // Tracks whether the "filler" content is shown
  showFiller = false;

  // Toggles the sidenav's open state and emits the new state
    toggleSideNav() {
    // Toggle the current state of the sidenav (open/closed)
    this.isOpen = !this.isOpen;

    // Emit the new state of the sidenav to notify parent components
    // This allows external components to react when the sidenav is toggled
    this.sidenavToggled.emit(this.isOpen);
    }

  // Toggles the visibility of the filler content
  toggleFiller() {
    this.showFiller = !this.showFiller;
  }

  // Listens for document click events
  @HostListener('document:click', ['$event'])
  onDocumentClick(event: MouseEvent) {
    // If the click target is not a valid element or matches certain conditions, exit early
    if (
      !(event.target instanceof Element) ||
      (event.target instanceof HTMLElement &&
        (event.target.tagName === 'BUTTON' ||
         event.target.classList.contains('mat-drawer-inner-container')))
    ) {
      return;
    }

    // If the sidenav is open and the conditions are met, toggle the sidenav
    if (this.isOpen) {
      this.toggleSideNav();
    }
  }
}
```

**HTML (`sidenav.component.html`)**  

```html
<!-- Main container for the sidenav -->
<div class="sidenav-container">
  
  <!-- Sidenav section that adjusts based on the open/closed state -->
  <div class="sidenav" [class.open]="isOpen">
    <!-- Static content for the sidenav -->
    <p>Auto-resizing sidenav</p>

    <!-- Filler content displayed conditionally based on the showFiller flag -->
    <p *ngIf="showFiller">Lorem, ipsum dolor sit amet consectetur.</p>

    <!-- Button to toggle the visibility of the filler content -->
    <button class="toggle-filler-button" (click)="toggleFiller()">
      Toggle extra text
    </button>
  </div>

  <!-- Sidenav content area that appears when the sidenav is closed -->
  <div class="sidenav-content" *ngIf="!isOpen">
    <!-- Button to toggle the sidenav open/closed -->
    <button class="toggle-sidenav-button" (click)="toggleSideNav()">Toggle sidenav</button>
  </div>

</div>

```

---

**(CSS (`sidenav.component.scss`)**)

```css
.sidenav-container {
  display: flex;
  flex-direction: row;
}

.sidenav {
  width: 250px;
  transition: transform 0.3s ease;
  background-color: #f4f4f4;
  padding: 1rem;
  border-right: 1px solid #ddd;
}

.sidenav:not(.open) {
  transform: translateX(-100%);
}

.sidenav-content {
  flex-grow: 1;
  padding: 1rem;
}

.toggle-filler-button,
.toggle-sidenav-button {
  background-color: #007bff;
  color: #fff;
  border: none;
  padding: 0.5rem 1rem;
  cursor: pointer;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}

.toggle-filler-button:hover,
.toggle-sidenav-button:hover {
  background-color: #0056b3;
}
```

---

#### **After Migration**  

Now let’s migrate this to use **Signals**. Signals replace the mutable properties `isOpen` and `showFiller` to make them reactive and easier to manage.

The migration to Angular Signals involves replacing the mutable properties (`isOpen` and `showFiller`) with reactive signals. Signals help in tracking and propagating changes more efficiently, enhancing performance and predictability in the application.

### **Key Changes**:
- **Signals** (`signal`) are used instead of mutable variables.
- **Effects** (`effect`) are employed to automatically emit changes when signals are updated.

Here’s a breakdown of the changes in the code:

#### **TypeScript (`sidenav.component.ts`)**

1. **Initialization**:
   - `isOpen` and `showFiller` are replaced with `signal` to manage state reactively.
   - The initial state is set via the constructor, passing the input value (`initialOpen`) to the `signal`.

2. **Effect for Emitting Changes**:
   - The `effect` hook listens for changes in `isOpen()` and emits the new state to the parent component whenever it changes.

3. **State Update**:
   - `toggleSideNav` now updates `isOpen` using the `update` method of `signal`, which flips the state.

```typescript
import { CommonModule } from '@angular/common';
import { Component, HostListener, Input, Output, EventEmitter, signal, effect } from '@angular/core';

@Component({
  selector: 'app-sidenav',
  imports: [CommonModule],
  templateUrl: './sidenav.component.html',
  styleUrls: ['./sidenav.component.scss']
})
export class SidenavComponent {
  @Input() initialOpen = false;
  @Output() sidenavToggled = new EventEmitter<boolean>();

  // Replacing mutable states with signals
  isOpen = signal<boolean>(this.initialOpen); 
  showFiller = signal<boolean>(false);

  constructor() {
    // Effect to emit the sidenav state when isOpen changes
    effect(() => {
      this.sidenavToggled.emit(this.isOpen()); 
    });
  }

  // Function to toggle the sidenav
  toggleSideNav() {
    this.isOpen.update((prev) => !prev); // Toggling the signal
  }

  // Function to toggle the filler content visibility
  toggleFiller() {
    this.showFiller.update((prev) => !prev); // Toggling the filler visibility signal
  }

  @HostListener('document:click', ['$event'])
  onDocumentClick(event: MouseEvent) {
    if (
      !(event.target instanceof Element) ||
      (event.target instanceof HTMLElement &&
        (event.target.tagName === 'BUTTON' ||
         event.target.classList.contains('mat-drawer-inner-container')))
    ) {
      return;
    }

    if (this.isOpen()) {
      this.toggleSideNav();
    }
  }
}
```

#### **HTML (`sidenav.component.html`)**

No significant change is required for the HTML part. It continues to display the dynamic state based on the signals (`isOpen` and `showFiller`). You can bind the signals in the same way as before, using the `{{ signal() }}` syntax to access their values:

```html
<div class="sidenav-container">
  <div class="sidenav" [class.open]="isOpen()">
    <p>Auto-resizing sidenav</p>
    <p *ngIf="showFiller()">Lorem, ipsum dolor sit amet consectetur.</p>
    <button class="toggle-filler-button" (click)="toggleFiller()">
      Toggle extra text
    </button>
  </div>

  <div class="sidenav-content" *ngIf="!isOpen()">
    <button class="toggle-sidenav-button" (click)="toggleSideNav()">Toggle sidenav</button>
  </div>
</div>
```

---
Angular 19 introduces the `ng update` command, which includes built-in support for migrating existing code to use the Signals API. This built-in tool can automatically help with the migration, making it much easier to transition your application to use Signals.

Here's how you can use it to migrate your code:

### **Migrating to Signals using Angular CLI in Angular 19+**

#### **Step-by-Step Process**:

1. **Update Angular to the latest version**:
   
   Ensure you're using Angular 19 or later, as the migration tool is available starting from this version.
   ```bash
   ng update @angular/cli @angular/core
   ```

2. **Review the Migration Report**:

   After running the migration, Angular will generate a detailed report summarizing the changes made. This will include:
   - Which state properties were replaced with `signal()`.
   - Any derived values that were changed to use `computed()`.
   - Side effects that were refactored into `effect()`.

   You can review this report to ensure everything is correctly updated.

3. **Manual Review (if needed)**:
   
   While most of the migration will be automated, it's a good idea to manually review any custom logic to ensure that signals, computed values, and effects are applied in the most optimal way for your application.

## **Wrong Commands**:

```bash
ng update @angular/core -signals
```

```bash
ng update @angular/core --signals
```

```bash
ng generate @angular/core signals
```

```bash
ng @angular/core signals
```

### **Previous Command**:

```bash
ng update @angular/core --migrate-signals
```

## **Angular v19 correct command**:

```bash
ng generate @angular/core:signals
```

You can choose to use the above interactive way or choose specific parts to migrate:

Certainly, let's break down how to migrate `@Input`, `@Output`, and queries to use Signals in Angular.

**1. Migrating `@Input`**

* **`ng generate @angular/core:signal-input-migration`**

This command helps you migrate `@Input` properties to use Signals. 

* **How it works:**
    - The command analyzes your component's `@Input` properties.
    - It creates a `signal` for each `@Input` property.
    - It updates the component's template to use the `signal()` method to access the input values.

* **Example:**

    **Before:**

    ```typescript
    @Component({ 
      // ...
    })
    export class MyComponent {
      @Input() myInput: string; 
      // ...
    }
    ```

    **After:**

    ```typescript
    @Component({ 
      // ...
    })
    export class MyComponent {
      myInput = signal<string>(''); 

      // ...
    }
    ```

**2. Migrating `@Output`**

* **`ng generate @angular/core:output-migration`**

This command helps you migrate `@Output` properties to use Signals.

* **How it works:**
    - The command analyzes your component's `@Output` properties.
    - It creates a `signal` to represent the emitted value.
    - It updates the component's logic to update the signal when an event occurs.

* **Example:**

    **Before:**

    ```typescript
    @Component({ 
      // ...
    })
    export class MyComponent {
      @Output() myEvent = new EventEmitter<string>(); 
      // ...
    }
    ```

    **After:**

    ```typescript
    @Component({ 
      // ...
    })
    export class MyComponent {
      myEvent = signal<string>(''); 

      emitMyEvent(value: string) {
        this.myEvent.set(value); 
      }
      // ...
    }
    ```

**3. Migrating Queries**

* **`ng generate @angular/core:signal-queries-migration`**

This command helps you migrate queries (like `@ViewChild`, `@ViewChildren`, `@ContentChild`, `@ContentChildren`) to use Signals.

* **How it works:**
    - The command analyzes your component's queries.
    - It creates a `signal` to hold the queried elements.
    - It updates the component's logic to update the signal when the queried elements change.

* **Example:**

    **Before:**

    ```typescript
    @Component({ 
      // ...
    })
    export class MyComponent {
      @ViewChild('myElement') myElementRef: ElementRef; 
      // ...
    }
    ```

    **After:**

    ```typescript
    @Component({ 
      // ...
    })
    export class MyComponent {
      myElementRef = signal<ElementRef | null>(null); 
      // ...
    }
    ```

**Key Considerations:**

* **Migration Reports:** Each migration command generates a report that summarizes the changes made to your code. Review these reports carefully to ensure the migration was successful.
* **Manual Review:** After running the migration commands, it's highly recommended to manually review the generated code to ensure it functions as expected and aligns with your application's specific requirements.
* **Testing:** Thoroughly test your component after migration to ensure that all functionality is working as expected.

By using these commands and carefully reviewing the generated code, you can effectively migrate your `@Input`, `@Output`, and queries to use Signals in your Angular application.

I hope this explanation is more helpful! Let me know if you have any further questions.

### **Benefits of Migrating to Signals**:
1. **Improved Performance**: With Signals, Angular optimizes the update cycle, only updating the DOM when necessary based on signal changes.
2. **Predictable State**: The state is managed reactively, making it easier to reason about the application's behavior and debug.
3. **Declarative Code**: Signals simplify how state changes are handled, aligning with modern reactive programming paradigms and reducing the need for complex state management solutions.

By migrating to Signals, the component benefits from better reactivity management, improved readability and enhanced performance.

### **Conclusion**  

Angular’s migration to Signals represents a bold step towards modernizing its reactivity model. By embracing this paradigm shift, Angular is not only addressing performance bottlenecks but also simplifying how developers manage state.

With Angular's migration to Signals, managing state becomes more intuitive and performant, allowing developers to focus more on logic rather than intricate change detection mechanisms. The adoption of Signals represents a forward-thinking shift towards a cleaner and more predictable reactive programming model.

---  

Would you like to explore more about migrating an existing Angular app to use Signals or dive into practical examples? Let me know!

---

Follow me on Github to get the latest updates: [Adioz](https://x.com/adiozdaniel/)

