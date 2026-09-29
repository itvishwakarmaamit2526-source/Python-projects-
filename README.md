"""
MVLU COLLEGE
EXAMINATION HALL SEAT ALLOCATION MANAGEMENT SYSTEM

A beginner-friendly Tkinter project demonstrating:
- Python basics, variables, constants and data types
- Control flow (if, for, while)
- Lists, dictionaries and string operations
- Functions and standard-library modules
- CSV file operations
- Regular expressions for input validation
- Tkinter GUI widgets and event handling
"""

import csv
import re
import tkinter as tk
from tkinter import ttk, messagebox

# Constants and in-memory data structures
COLLEGE_NAME = "MVLU COLLEGE"
HALL_NO = "HALL-01"
TOTAL_SEATS = 30
DATA_FILE = "seat_allocations.csv"

# Each student is stored as a dictionary inside the students list.
students = []


def clean_name(name):
    """Remove extra spaces and format a student's name."""
    return " ".join(name.strip().split()).title()


def clean_roll(roll):
    """Remove surrounding spaces and standardize roll number."""
    return roll.strip().upper()


def valid_student_name(name):
    """Allow alphabetic names with spaces, dots, apostrophes or hyphens."""
    return re.fullmatch(r"[A-Za-z][A-Za-z .'-]{1,38}", name) is not None


def valid_roll_number(roll):
    """Allow 2-12 letters, digits or hyphens in a roll number."""
    return re.fullmatch(r"[A-Z0-9-]{2,12}", roll) is not None


def load_allocations():
    """Read previously saved allocations from a CSV file."""
    loaded = []

    try:
        with open(DATA_FILE, "r", newline="", encoding="utf-8") as file:
            reader = csv.DictReader(file)

            for row in reader:
                # Control flow checks each row before adding it.
                if row.get("name") and row.get("roll") and row.get("seat"):
                    loaded.append({
                        "name": row["name"],
                        "roll": row["roll"],
                        "seat": row["seat"]
                    })

    except FileNotFoundError:
        # First run: no data file exists yet.
        pass
    except OSError as error:
        messagebox.showwarning(
            "File Warning", f"Could not read saved data:\n{error}"
        )

    return loaded[:TOTAL_SEATS]


def save_allocations():
    """Save all student records to a CSV file."""
    with open(DATA_FILE, "w", newline="", encoding="utf-8") as file:
        fieldnames = ["name", "roll", "seat"]
        writer = csv.DictWriter(file, fieldnames=fieldnames)
        writer.writeheader()
        writer.writerows(students)


def find_student(roll):
    """Find a student by roll number; return a dictionary or None."""
    for student in students:
        if student["roll"] == roll:
            return student
    return None


def allocate_seat(name, roll):
    """Validate details, assign the next seat and save the record."""
    name = clean_name(name)
    roll = clean_roll(roll)

    if not valid_student_name(name):
        return False, "Enter a valid name using letters and normal punctuation."

    if not valid_roll_number(roll):
        return False, "Roll number must be 2-12 letters, digits or hyphens."

    if find_student(roll) is not None:
        return False, "This roll number is already allocated a seat."

    if len(students) >= TOTAL_SEATS:
        return False, "The examination hall is full."

    seat_number = len(students) + 1
    seat = f"{HALL_NO}-{seat_number:02d}"

    record = {"name": name, "roll": roll, "seat": seat}
    students.append(record)

    try:
        save_allocations()
    except OSError as error:
        students.pop()
        return False, f"Could not save the record:\n{error}"

    return True, f"Seat allocated successfully: {seat}"


def search_student(roll):
    """Return a matching student's details for the search action."""
    return find_student(clean_roll(roll))


def refresh_table():
    """Refresh the GUI table and seat availability label."""
    for item in table.get_children():
        table.delete(item)

    for student in students:
        table.insert(
            "", tk.END,
            values=(student["name"], student["roll"], student["seat"])
        )

    available = TOTAL_SEATS - len(students)
    availability_label.config(
        text=f"Available Seats: {available} / {TOTAL_SEATS}"
    )


def on_allocate():
    """Handle the Allocate Seat button click."""
    success, result = allocate_seat(name_entry.get(), roll_entry.get())

    if success:
        messagebox.showinfo("Allocation", result)
        name_entry.delete(0, tk.END)
        roll_entry.delete(0, tk.END)
        refresh_table()
    else:
        messagebox.showerror("Unable to Allocate", result)


def on_search():
    """Handle the Search button click."""
    roll = clean_roll(search_entry.get())

    if not roll:
        messagebox.showwarning("Search", "Please enter a roll number.")
        return

    student = search_student(roll)

    if student:
        messagebox.showinfo(
            "Student Found",
            f"Name: {student['name']}\n"
            f"Roll No: {student['roll']}\n"
            f"Seat: {student['seat']}"
        )
    else:
        messagebox.showinfo("Search", "No student found for this roll number.")


def on_show_all():
    """Display all currently loaded student records."""
    refresh_table()
    if not students:
        messagebox.showinfo("Records", "No seat allocations available yet.")


def create_app():
    """Build the Tkinter window, widgets and event bindings."""
    global root, table, availability_label
    global name_entry, roll_entry, search_entry

    root = tk.Tk()
    root.title(f"{COLLEGE_NAME} - Seat Allocation")
    root.geometry("760x540")
    root.minsize(620, 440)
    root.configure(bg="#f2f5f9")

    heading = tk.Label(
        root,
        text=f"{COLLEGE_NAME}\nEXAMINATION HALL SEAT ALLOCATION",
        font=("Arial", 17, "bold"),
        bg="#f2f5f9",
        fg="#17365d",
        justify="center"
    )
    heading.pack(pady=(18, 6))

    subheading = tk.Label(
        root,
        text=f"Hall No: {HALL_NO}",
        font=("Arial", 11),
        bg="#f2f5f9"
    )
    subheading.pack(pady=(0, 10))

    form = ttk.LabelFrame(root, text="Allocate New Seat", padding=12)
    form.pack(fill="x", padx=20, pady=8)

    ttk.Label(form, text="Student Name:").grid(
        row=0, column=0, sticky="w", padx=5, pady=6
    )
    name_entry = ttk.Entry(form, width=28)
    name_entry.grid(row=0, column=1, padx=5, pady=6, sticky="ew")

    ttk.Label(form, text="Roll Number:").grid(
        row=0, column=2, sticky="w", padx=5, pady=6
    )
    roll_entry = ttk.Entry(form, width=18)
    roll_entry.grid(row=0, column=3, padx=5, pady=6, sticky="ew")

    allocate_button = ttk.Button(
        form, text="Allocate Seat", command=on_allocate
    )
    allocate_button.grid(
        row=1, column=0, columnspan=4, padx=5, pady=(8, 2)
    )
    form.columnconfigure(1, weight=1)
    form.columnconfigure(3, weight=1)

    search_frame = ttk.LabelFrame(root, text="Search Student", padding=10)
    search_frame.pack(fill="x", padx=20, pady=8)

    search_entry = ttk.Entry(search_frame)
    search_entry.pack(side="left", fill="x", expand=True, padx=(0, 8))
    ttk.Button(
        search_frame, text="Search by Roll No", command=on_search
    ).pack(side="left")
    ttk.Button(
        search_frame, text="Show All", command=on_show_all
    ).pack(side="left", padx=(8, 0))

    availability_label = tk.Label(
        root,
        text="",
        font=("Arial", 11, "bold"),
        bg="#f2f5f9",
        fg="#1b6b3a"
    )
    availability_label.pack(pady=7)

    columns = ("name", "roll", "seat")
    table = ttk.Treeview(root, columns=columns, show="headings", height=11)
    table.heading("name", text="Student Name")
    table.heading("roll", text="Roll Number")
    table.heading("seat", text="Seat Number")
    table.column("name", width=280)
    table.column("roll", width=170, anchor="center")
    table.column("seat", width=170, anchor="center")
    table.pack(fill="both", expand=True, padx=20, pady=(0, 12))

    footer = tk.Label(
        root,
        text="Student records are stored locally in seat_allocations.csv",
        font=("Arial", 9),
        bg="#f2f5f9",
        fg="#555555"
    )
    footer.pack(pady=(0, 10))

    root.bind("<Return>", lambda event: on_allocate())
    refresh_table()
    return root


if __name__ == "__main__":
    students = load_allocations()
    app = create_app()
    app.mainloop()
