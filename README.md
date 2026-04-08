# 🏥 Clinic Appointment and Diagnostics Platform — ER Diagram

A database design project for a Clinic Appointment and Diagnostics Platform, submitted as part of an academic assignment. This repository contains the Entity-Relationship Diagram (ERD) representing the full data model of a clinic system — from patient registration to appointments, consultations, diagnostics, test results, reports, and payments.

---

## 📌 Project Overview

This design models a real-world clinic workflow:

> Patient books appointment → Doctor is assigned → Consultation happens → Tests are prescribed → Results are recorded → Report is generated → Payment is collected

The goal was to design a **clean, scalable, and normalized** database schema — not to build a full hospital system, but a focused clinic platform handling:

- Appointments & Scheduling
- Doctor-Patient Consultations
- Diagnostic Test Management
- Test Results & Medical Reports
- Payment Tracking

---

## 🗂️ Entity Summary

| Entity | Description |
|---|---|
| `patient` | Stores patient personal and medical info |
| `doctor` | Stores doctor profile and clinic joining details |
| `doctor_qualification` | Stores multiple qualifications per doctor |
| `department` | Clinic departments with HOD reference |
| `appointment` | Booking record linking patient, doctor, department |
| `consultation` | Actual visit/meeting record with fee and notes |
| `test_catalog` | Master list of available diagnostic tests with pricing |
| `consultation_test` | Junction table — tests prescribed during a consultation |
| `test_results` | Raw test result values and report files |
| `reports` | Doctor's diagnosis and interpretation after consultation |
| `payments` | Payment record with fee snapshots and payment method |

---

## 🔗 Relationship Cardinality Table

| From Entity | To Entity | Cardinality | Notes |
|---|---|---|---|
| `department` | `doctor` | One to Many | One department has many doctors |
| `doctor` | `department` | One to One | One doctor belongs to one department |
| `department` | `appointment` | One to Many | Appointments are booked under a department |
| `patient` | `appointment` | One to Many | One patient can have many appointments |
| `doctor` | `appointment` | One to Many | One doctor can have many appointments |
| `appointment` | `consultation` | One to Many | One appointment can lead to multiple consultations (follow-ups) |
| `doctor` | `consultation` | One to Many | One doctor can conduct many consultations |
| `patient` | `consultation` | One to Many | One patient can have many consultations |
| `consultation` | `consultation_test` | One to Many | One consultation can prescribe many tests |
| `test_catalog` | `consultation_test` | One to Many | One catalog item can appear in many consultations |
| `consultation_test` | `test_results` | One to Many | One prescribed test can have many result entries |
| `consultation` | `reports` | One to One | One consultation generates one report |
| `consultation` | `payments` | One to One | One consultation has one payment record |
| `doctor` | `doctor_qualification` | One to Many | One doctor can have multiple qualifications |

---

## 🔀 Junction Tables

This design uses **one junction table** to resolve a Many-to-Many relationship:

### `consultation_test`
| Column | Type | Role |
|---|---|---|
| `consultation_test_id` | INT | Primary Key |
| `consultation_id` | INT FK | References `consultation` |
| `catalog_id` | INT FK | References `test_catalog` |
| `notes` | TEXT | Additional prescription notes |
| `status` | ENUM | Current status of the prescribed test |
| `prescribed_at` | TIMESTAMP | When the test was prescribed |

**Why this junction table exists:**
- A single **consultation** can prescribe **many tests**
- A single **test from the catalog** can be prescribed in **many consultations**
- This is a classic Many-to-Many relationship — `consultation_test` sits in between and resolves it
- It also carries its own attributes (`notes`, `status`, `prescribed_at`) making it a rich junction table, not just a linking table

---

## 🧠 Design Decisions

- **Appointment vs Consultation**: An appointment is a *booking*. A consultation is the *actual visit*. They are kept separate because an appointment may be canceled, rescheduled, or result in multiple follow-up consultations.

- **Test Catalog vs Consultation Test**: `test_catalog` is a master reference list of tests. `consultation_test` is the junction table representing a specific test prescribed to a patient in a specific consultation.

- **Price Snapshots in Payments**: Prices are stored as snapshots at the time of payment to prevent historical data corruption when clinic prices change.

- **Doctor Availability**: Derived from appointments — no separate availability table needed. If a doctor has a `scheduled` appointment at a given time, they are considered busy.

- **`doctor_id` nullable in Appointments**: At booking time, a department is selected first. The specific doctor may be assigned later, so `doctor_id` is nullable.

- **`is_followup` flag in Consultation**: Differentiates a follow-up visit from a first-time consultation, allowing backend logic to apply reduced fees accordingly.

- **`is_emergency` flag in Appointment**: Differentiates walk-in emergencies from scheduled appointments.

---
