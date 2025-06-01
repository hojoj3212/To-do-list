import json
from datetime import datetime, timedelta
from enum import Enum, auto

class Priority(Enum):
    LOW = auto()
    MEDIUM = auto()
    HIGH = auto()

class Task:
    def __init__(self, description, category, due_date=None, priority=Priority.MEDIUM):
        self.description = description
        self.category = category
        self.due_date = due_date
        self.priority = priority
        self.completed = False
        self.created_at = datetime.now()

    def to_dict(self):
        return {
            "description": self.description,
            "category": self.category,
            "due_date": self.due_date.isoformat() if self.due_date else None,
            "priority": self.priority.name,
            "completed": self.completed,
            "created_at": self.created_at.isoformat()
        }

    @classmethod
    def from_dict(cls, data):
        task = cls(
            data["description"],
            data["category"],
            datetime.fromisoformat(data["due_date"]) if data["due_date"] else None,
            Priority[data["priority"]]
        )
        task.completed = data["completed"]
        task.created_at = datetime.fromisoformat(data["created_at"])
        return task

class TaskManager:
    def __init__(self, filename="tasks.json"):
        self.filename = filename
        self.tasks = self._load_tasks()

    def _load_tasks(self):
        try:
            with open(self.filename, "r") as f:
                return [Task.from_dict(task_data) for task_data in json.load(f)]
        except (FileNotFoundError, json.JSONDecodeError):
            return []

    def _save_tasks(self):
        with open(self.filename, "w") as f:
            json.dump([task.to_dict() for task in self.tasks], f, indent=2)

    def add_task(self, description, category, due_date=None, priority=Priority.MEDIUM):
        self.tasks.append(Task(description, category, due_date, priority))
        self._save_tasks()

    def edit_task(self, task_id, **kwargs):
        if 0 <= task_id < len(self.tasks):
            task = self.tasks[task_id]
            for key, value in kwargs.items():
                if hasattr(task, key):
                    setattr(task, key, value)
            self._save_tasks()

    def complete_task(self, task_id):
        self.edit_task(task_id, completed=True)

    def delete_task(self, task_id):
        if 0 <= task_id < len(self.tasks):
            del self.tasks[task_id]
            self._save_tasks()

    def get_tasks(self, filter_func=None):
        return filter(filter_func, self.tasks) if filter_func else self.tasks

    def get_overdue_tasks(self):
        return self.get_tasks(lambda t: t.due_date and t.due_date < datetime.now() and not t.completed)

    def get_tasks_by_category(self, category):
        return self.get_tasks(lambda t: t.category == category)

    def get_tasks_by_priority(self, priority):
        return self.get_tasks(lambda t: t.priority == priority)

def display_tasks(tasks):
    if not tasks:
        print("No tasks found.")
        return

    for i, task in enumerate(tasks):
        status = "✓" if task.completed else "✗"
        due = f" (Due: {task.due_date.strftime('%Y-%m-%d')})" if task.due_date else ""
        print(f"{i}. [{status}] {task.description} [{task.category}] {due} ({task.priority.name})")

def input_date(prompt):
    while True:
        date_str = input(prompt + " (YYYY-MM-DD, leave empty if none): ")
        if not date_str:
            return None
        try:
            return datetime.strptime(date_str, "%Y-%m-%d")
        except ValueError:
            print("Invalid date format. Try again.")

def input_priority():
    print("Priority:")
    for i, priority in enumerate(Priority, 1):
        print(f"{i}. {priority.name}")
    while True:
        try:
            choice = int(input("Select priority: "))
            if 1 <= choice <= len(Priority):
                return list(Priority)[choice - 1]
        except ValueError:
            print("Invalid input. Enter a number.")

def main():
    manager = TaskManager()

    while True:
        print("\n1. Add Task\n2. View Tasks\n3. Edit Task\n4. Complete Task\n5. Delete Task")
        print("6. View Overdue Tasks\n7. View by Category\n8. View by Priority\n9. Exit")
        choice = input("Choose an option: ")

        if choice == "1":
            desc = input("Task description: ")
            category = input("Category: ")
            due_date = input_date("Due date")
            priority = input_priority()
            manager.add_task(desc, category, due_date, priority)

        elif choice == "2":
            display_tasks(manager.get_tasks())

        elif choice == "3":
            display_tasks(manager.get_tasks())
            try:
                task_id = int(input("Enter task number to edit: "))
                desc = input(f"New description (leave empty to keep): ")
                category = input(f"New category (leave empty to keep): ")
                due_date = input_date("New due date")
                priority = input_priority() if input("Change priority? (y/n): ").lower() == "y" else None
                updates = {}
                if desc: updates["description"] = desc
                if category: updates["category"] = category
                if due_date is not None: updates["due_date"] = due_date
                if priority: updates["priority"] = priority
                manager.edit_task(task_id, **updates)
            except (ValueError, IndexError):
                print("Invalid task number.")

        elif choice == "4":
            display_tasks(manager.get_tasks())
            try:
                task_id = int(input("Enter task number to complete: "))
                manager.complete_task(task_id)
            except (ValueError, IndexError):
                print("Invalid task number.")

        elif choice == "5":
            display_tasks(manager.get_tasks())
            try:
                task_id = int(input("Enter task number to delete: "))
                manager.delete_task(task_id)
            except (ValueError, IndexError):
                print("Invalid task number.")

        elif choice == "6":
            display_tasks(manager.get_overdue_tasks())

        elif choice == "7":
            category = input("Enter category: ")
            display_tasks(manager.get_tasks_by_category(category))

        elif choice == "8":
            priority = input_priority()
            display_tasks(manager.get_tasks_by_priority(priority))

        elif choice == "9":
            break

        else:
            print("Invalid option.")

if __name__ == "__main__":
    main()
