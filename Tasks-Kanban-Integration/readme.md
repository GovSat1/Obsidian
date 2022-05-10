# Obsidian Tasks Kanban Integration

Modify Tasks plugin to improve workflow with Kanban board dates.

This is a "hacky" solution and may break on future Tasks plugin updates (currently done on Tasks v1.5.1). Hopefully the author will add support for custom signifiers soon.

Once the changes have been made:
- Tasks can be created from the Kanban board and they will show up as tasks automatically with dates
- Dates for a task can be changed from either Tasks or Kanban plugin
- All dated tasks from a Kanban board can be automatically tagged with `#task` (optional)
- Times (from Kanban) are ignored but retained for display in tasks
- Recurrence will show up as an emoji only (the rule is hidden with using markdown comments)
- Only due dates are modified here, but other dates can be modified in the same manner

**Result**

Kanban Board

![1](https://user-images.githubusercontent.com/36019883/167658893-508dbe9a-d458-4655-a46e-24e4d978c519.png)

Tasks Query Block

![2](https://user-images.githubusercontent.com/36019883/167658939-c35679b2-84af-4bb6-820f-4e6a8c887e36.png)

## Process

### Kanban Settings

1. In either the board or global Kanban settings, set the date format to `\#\t\a\s\k 📅 YYYY-MM-DD` or `📅 YYYY-MM-DD` (no automatic tag)
3. Optionally, change the date display format to something else, IE `YYYY-MM-DD` which will hide the emoji and tag from the card
4. Optionally, disable "Add date/time to archived cards". Tasks already does this with the "done" date format/emoji and it creates unnecessary noise when viewing completed tasks in a query.

### Tasks Plugin Settings

1. Set global filter to `#task`
2. Check "Remove global filter from Description"

### Modify the Tasks main.js file

This is located at: `your vault folder/plugins/obsidian-tasks-plugin/main.js`. Make a copy before proceding.

#### 1. Fix Kanban bracket formatting

Replace
```
      if (removeGlobalFilter) {
        taskAsString = taskAsString.replace(globalFilter, "").trim();
      }
```
with
```
      if (removeGlobalFilter) {
        taskAsString = taskAsString.replace(globalFilter, "").trim();
        taskAsString = taskAsString.replace(/@@\{|@\{|\}/g, "").trim();
      }
```

> This uses the function that removes the `#task` tag from display view to also remove the Kanban date and time formatters `@{}` and `@@{}`

#### 2. Fix Regex Date Parser

Replace   
```
Task.dueDateRegex = /[📅📆🗓] ?(\d{4}-\d{2}-\d{2})$/u;
```
with
```
Task.dueDateRegex = /@\{(?:#task\s)?[📅📆🗓] ?(\d{4}-\d{2}-\d{2})\}/u;
```
> This will parse Kanban dates formatted as either `@{#task 📅 2022-01-01}` or `@{📅 2022-01-01}`

#### 3. Fix Tasks Modify/Create Output Format

Replace
```
    if (!layoutOptions.hideDueDate && this.dueDate) {
      const dueDate = layoutOptions.shortMode ? " \u{1F4C5}" : ` \u{1F4C5} ${this.dueDate.format(_Task.dateFormat)}`;
      taskString += dueDate;
    }
```

with:

Option 1 -  automatically include a hidden tag with all tasks created from Task plugin
```
    if (!layoutOptions.hideDueDate && this.dueDate) {
      const dueDate = layoutOptions.shortMode ? " \u{1F4C5}" : ` @\{#task \u{1F4C5} ${this.dueDate.format(_Task.dateFormat)}\}`;
      taskString += dueDate;
    }
```

or Option 2 - do not include a hidden tag
```
    if (!layoutOptions.hideDueDate && this.dueDate) {
      const dueDate = layoutOptions.shortMode ? " \u{1F4C5}" : ` @\{\u{1F4C5} ${this.dueDate.format(_Task.dateFormat)}\}`;
      taskString += dueDate;
    }
```

> These options control the output format when a task is modified/created *from the Tasks plugin*

*If you chose option 1 also do:*
   
Replace:
```
	if (!description.includes(globalFilter)) {
      description = globalFilter + " " + description;
    }
```

with 
```
 /* if (!description.includes(globalFilter)) {
      description = globalFilter + " " + description;
    } */
```

> This disables appending the `#task` tag to the front of the task when it is modified/created *from the Tasks plugin*

#### 4. Fix recurrence rule format
   
   Replace 
```
    if (!layoutOptions.hideRecurrenceRule && this.recurrence) {
      const recurrenceRule = layoutOptions.shortMode ? " \u{1F501}" : ` \u{1F501} ${this.recurrence.toText()}`;
      taskString += recurrenceRule;
    }
```

with
```
    if (!layoutOptions.hideRecurrenceRule && this.recurrence) {
      const recurrenceRule = layoutOptions.shortMode ? " \u{1F501}" : ` \u{1F501} %%${this.recurrence.toText()}%%`;
      taskString += recurrenceRule;
    }
```

**And**

Replace
```
Task.recurrenceRegex = /🔁 ?([a-zA-Z0-9, !]+)$/iu;
```

with
```
Task.recurrenceRegex = /🔁 ?%?%?([a-zA-Z0-9, !]+)%?%?$/iu;
```

> This formats the recurrence rule to display as `🔁 %%every month%%`, which results in only the recurrence emoji displaying on the Kanban card


## Kanban CSS Changes

Optionally, add a CSS snippet as follows, to strikethrough completed tasks in Kanban board view.

```css
.kanban-plugin__item.is-complete{text-decoration:line-through;}
```
