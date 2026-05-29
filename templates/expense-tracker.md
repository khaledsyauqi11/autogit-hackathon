---
title: Personal Expense Tracker
app_type: personal-expense-tracker
wallet: 0x7d8369c99339e1b9fd1087db50d24926ed06095c
---

You are building a Personal Expense Tracker app called "SpendWise" using React + Vite + Tailwind CSS. Generate a complete, fully functional single-page application.

## App Requirements

### Add Expense Form
- Fields: Amount (number input), Category (dropdown: Food, Transport, Shopping, Health, Entertainment, Other), Note (text input), Date (date picker, default today)
- Submit button labeled "Add Expense"
- Validate: amount must be > 0 before submitting

### Expense List
- Show all expenses sorted by date (newest first)
- Each row: category icon (emoji), note, date, amount with currency symbol ($)
- Delete button (×) on each row
- Empty state message: "No expenses yet. Start tracking!"

### Summary Dashboard at the top
- Total Spent This Month (large number)
- Three top-category tiles showing category name, emoji, and total amount
- A simple bar chart (built with plain CSS flex bars, no library) showing spending per category

### Data Persistence
- Save all expenses to localStorage
- Load from localStorage on app start

### Visual Design
- Clean white card layout on light gray (#f8fafc) background
- Green accent (#10b981) for positive/summary elements
- Red accent (#ef4444) for delete and high-spend indicators
- Font: "Nunito" from Google Fonts
- Smooth slide-in animation when a new expense is added

### Technical Rules
- Single App.jsx component
- No external chart libraries — use CSS only for bars
- No backend required
