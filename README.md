# Vue Form Submission App

This is a simple Vue 3 form submission application that demonstrates how to create a dynamic form with validation, using PrimeVue components like `Textarea`, `RadioButton`, `Dropdown`, and `InputText`. The form allows users to input a description, select VAT, input the price netto, and dynamically calculate the price brutto. It includes error handling and validation for user inputs.

## Features

- **Dynamic Form Inputs**:
    - Description field with character limit.
    - Radio buttons for sending confirmation.
    - Dropdown for selecting VAT.
    - Price netto input field, dynamically enabled when a VAT is selected, supporting commas and dots as decimal separators.
    - Dynamic price brutto calculation based on VAT and price netto inputs.

- **Real-Time Validation**:
    - Validates form fields on the fly and prevents submission when the input is invalid.
    - Error messages are displayed beside the fields without affecting the layout.

- **Submission Simulation**:
    - Simulates form submission to an external API and displays a success message upon submission.

## Technologies Used

- **Vue 3**: Frontend framework for building the app.
- **PrimeVue**: UI component library for Vue.js used for form inputs and buttons.
- **Vite**: Used for building and running the development server.
- **Tailwind CSS**: For styling the components and layout.

## Requirements

- **Node.js** (v14 or higher)
- **npm** (or **yarn**) for managing dependencies.

## Installation

1. Go to client directory (``cd client``) from root
2. Run ``npm i``
3. Run ``npm run dev ``
4. App is ready on ``localhost:5173``
