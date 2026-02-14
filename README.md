# 🚀 ServiceNow Automation: Chelsea Stadium Puzzle Fulfillment

## 📑 Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [File Inventory](#file-inventory)
4. [Phase 1: Catalog Item Configuration](#phase-1-catalog-item-configuration)
5. [Phase 2: Flow Designer Logic (Step-by-Step)](#phase-2-flow-designer-logic-step-by-step)
6. [How to Install](#how-to-install)
7. [Technical Takeaways for Recruiters](#technical-takeaways-for-recruiters)

---

## 🏛️ Project Overview
This project demonstrates an end-to-end ITSM (IT Service Management) automation. It replaces a manual ordering process with a digital workflow that handles:
* **Dynamic Pricing:** Automatic price adjustments based on puzzle size.
* **Managerial Gatekeeping:** Automated approval routing via dot-walking.
* **Conditional Fulfillment:** Specific tasks (like Gift Wrapping) only trigger based on user input.
* **Proactive Communication:** Automated email touchpoints throughout the lifecycle.

---

## 🏛️ System Architecture
The workflow follows a "Request-to-Fulfillment" path:
1. **Submission:** User selects puzzle options via the Service Portal.
2. **Approval:** Flow identifies the user's manager and pauses for their digital signature.
3. **Logic Branching:** If approved, the flow extracts variables; if rejected, it closes the request.
4. **Tasking:** Sequential tasks ensure inventory is checked before shipping.

---

## 📂 File Inventory
* [Puzzle_Fulfillment_Update_Set.xml](./Puzzle_Fulfillment_Update_Set.xml) - Complete technical package for import into ServiceNow.
* [Technical_Manual.txt](./Technical_Manual.txt) - Detailed, click-by-click build guide for manual replication.
* [Workflow Steps.PNG](./Workflow%20Steps.PNG) - Full visual map of the Flow Designer logic and branching.

---

## 📋 Phase 1: Catalog Item Configuration
**Navigation:** `Service Catalog > Catalog Definitions > Maintain Items`

| Field | Value |
| :--- | :--- |
| **Name** | Chelsea Stadium Puzzle |
| **Price** | $25.00 |
| **Flow** | Chelsea Puzzle Fulfillment (Attached after Phase 2) |

### **Variables Configuration**
| Type | Label | Name | Logic |
| :--- | :--- | :--- | :--- |
| **Multiple Choice** | Select Puzzle Size | `size` | Small ($0), Medium (+$12.99), Large (+$21.99) |
| **Yes/No** | Gift Wrapped? | `gift_wrapped` | Triggers conditional wrapping task |
| **Date** | Delivery Date | `date_of_delivery` | Mandatory; used in shipping emails |
| **Multi-line Text** | Delivery Instructions | `delivery_instructions` | Maps to fulfillment task |

---

## 🧠 Phase 2: Flow Designer Logic (Step-by-Step)

**TRIGGER:** Service Catalog

### **Step 1: Send Email (Acknowledgement)**
* **To:** `[Trigger->RITM->Requested for->Email]`
* **Subject:** Order Received - `[Trigger->RITM->Number]`
* **Body:** Hello, thank you for your order. Your request is pending manager approval.

### **Step 2: Ask For Approval**
* **Approver:** `[Trigger->RITM->Requested for->Manager]`
* **Rules:** Anyone Approves.

### **Step 3: FLOW LOGIC - IF (Branch 1)**
* **Condition:** `[Step 2 -> Approval State]` IS **Approved**

> **IF APPROVED BRANCH:**
>
> * **Step 4: Get Catalog Variables** (Extract: size, gift_wrap, date, instructions)
> * **Step 5: Send Email (Approved)**
>   * **Body:** Your manager has approved your request! We are now preparing your item.
> * **Step 6: Create Catalog Task (Inventory)**
>   * **Short Description:** Check stock for `[Step 4 -> Size]` puzzle.
> * **Step 7: FLOW LOGIC - IF (Nested)**
>   * **Condition:** `[Step 4 -> Gift Wrapped]` IS **True**
>     * **Step 8: Create Catalog Task:** "Gift wrap the puzzle."
> * **Step 9: Create Catalog Task (Shipping)**
>   * **Description:** `[Step 4 -> Delivery Instructions]`
> * **Step 10: Send Email (Shipped)**
>   * **Body:** Your order is on the way! Instructions: `[Step 4 -> Instructions]`
> * **Step 11: Update Record (RITM)**
>   * **State:** Closed Complete.

### **Step 12: FLOW LOGIC - ELSE (Branch 2)**
* **Note:** *Aligned with Step 3 (Rejected Path)*

> **IF REJECTED BRANCH:**
>
> * **Step 13: Update Record (RITM)**
>   * **State:** Closed Incomplete.
>   * **Work Notes:** "Manager has rejected this request."
> * **Step 14: Send Email (Rejection)**
>   * **Body:** Your request was not approved. It has been closed.

---

## 🛠️ How to Install
1. Download `Puzzle_Fulfillment_Update_Set.xml`.
2. In ServiceNow, navigate to **System Update Sets > Retrieved Update Sets**.
3. Import XML, **Preview**, and **Commit**.
4. Navigate to the **Chelsea Stadium Puzzle** item and click **Try It** to test.

---

## 💡 Key Functions
* **Dot-Walking:** Dynamically pulled manager data via reference fields.
* **Variable Mapping:** Successfully orchestrated data flow between Catalog Items and Flow Designer.
* **Error Handling:** Implemented an `Else` path to ensure terminal states for rejected records.
* **User Experience:** Integrated automated emails at every business milestone.
