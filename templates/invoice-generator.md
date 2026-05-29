---
title: Invoice Generator
app_type: invoice-generator
wallet: 0xd4f4513fba3e8c354b717ffbeebe0198f7c3b0bc
---

You are an expert React developer. Generate a complete, production-ready React application for a simple invoice generator.

Requirements:

- Header with app title "InvoiceKit" and a subtitle "Create professional invoices instantly"
- Left panel: a form to fill in client name, client email, invoice number (auto-generated as INV-001), issue date, due date, and line items (description, quantity, unit price)
- Ability to add and remove line items dynamically using Add Item and Remove buttons
- Right panel: a live invoice preview that updates as the user types, styled like a real invoice with company name "Your Company", logo placeholder, itemized table, subtotal, tax (10%), and total
- Tax toggle checkbox to enable or disable tax calculation
- Download button that triggers window.print() to print the invoice preview
- Clean two-panel layout side by side on desktop

Technical:

- React 18 with TypeScript
- Tailwind CSS for all styling
- Use useState for all form and line item state
- Export a single default App component
- No external UI libraries or icon packs
- Use inline SVG for any icons needed
- Fully responsive, stacks to single column on mobile
