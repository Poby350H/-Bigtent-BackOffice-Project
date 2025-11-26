# -Bigtent-BackOffice-Project

Shopify-Integrated Commerce Operations Backoffice
(Django Template + HTMX + Alpine.js + Bootstrap)

0. One-line Summary

I designed and implemented a backoffice from scratch that consolidates data collected from Shopify and Google Sheets into PostgreSQL, and uses a combination of Django Templates, HTMX, Alpine.js, and Bootstrap to manage inventory, sales, purchasing, and internal operations metrics in a single interface.

---

1. Project Overview

This project is an internal backoffice web application for managing Shopify store operations data in one place.

Product/inventory/order (sales) data from Shopify and
Safe Stock, purchase/ETA information organized in Google Sheets

are collected into PostgreSQL through a backend data pipeline.

On top of that, I built a Django-based backoffice that allows users to
view, filter, and analyze inventory, sales, purchasing, and internal operations data in a single interface.

---

2. Background & Problem

**There was no dedicated backoffice and no internal UI for operations.**

The team was running everything with a mix of Shopify admin + multiple Google Sheets/Excel files, managing: Inventory, sales/orders, POs, incoming quantities (ETAs)

Safe Stock wasn’t clearly defined or standardized: Everyone had a different idea of “safe” inventory, and many SKUs had no Safe Stock level at all.

As part of this project, I defined Safe Stock (e.g., coverage days, minimum units per SKU/brand) and systematized it by storing values in the DB and visualizing them in the backoffice.

It was hard to see “which SKU/brand is at risk right now” and staff had to jump between Shopify, Sheets, Excel, and email, doing a lot of manual filter/pivot/copy-paste work.

In short, operations were held together by manual work across Shopify and spreadsheets,
with no clear Safe Stock concept and no unified backoffice to consolidate and visualize everything.

---

3. Goals

•Build a dedicated backoffice for Shopify-based commerce operations

•Provide an internal tool that can manage
  inventory, sales, purchasing, incoming quantities, and Safe Stock in a single view.

•Fast list/search/filter UX

•Without full page reloads:
on filter/search/page changes, only the table area is partially updated.

•Maintainable structure without a heavy SPA framework

•Use Django Templates + HTMX + Alpine.js + Bootstrap to implement
  a “quasi-SPA” UX sufficient for admin screens without React/Next.js.

•Design that connects naturally to the data pipeline

•Clearly define the structure:
  Shopify/Google Sheets → (automation/ETL) → PostgreSQL → Django backoffice

---

4. Tech Stack
   
A.Backend

  Python, Django

  Django Templates (server-side rendering)

  PostgreSQL  In Google Could 
  
B.Frontend

  HTMX : On filter/page changes for inventory/order/PO lists, update only the table area, not the entire page.

  Alpine.js : Lightweight UI state management for modals, detail panels, filter panels, loading states, etc.

  Bootstrap 5 : Layout, grid system, tables, forms, button styling

C.Data Pipeline (Integration Layer – for explanation)

  Shopify API (REST/GraphQL)

  Google Sheets (gspread)

  Flask-based automation scripts/APIs
    → Regularly load and update Shopify/Sheets data into PostgreSQL.

---

5. Key Features 

  A.Inventory dashboard (Shopify-integrated)

    Sync Shopify inventory into PostgreSQL and show it by SKU/product/brand/location.

    Compare against Safe Stock and color-code Short / Warning / Normal.

    Sort by the most critical shortages first.

  B.Sales / Orders view

    ETL Shopify order data into a time-based sales list.

    Filter by date, country, channel, and brand.

    Use sales velocity per SKU/brand to prioritize purchasing.

  C.PO (Purchase Order) management

    List POs by supplier, PO number, ETA, and status.

    Calculate remaining quantity per PO (ordered - received).

    Decide what to reorder, when, and how much using inventory + sales + incoming data together.

  D.Search / filter / pagination with HTMX

    On filter/page changes, call partial endpoints and update only the table area.

    Keeps list navigation and filtering fast without full page reloads.

---

6. Data Pipeline & Architecture
   
•Data pipeline

  Shopify + Google Sheets → Flask-based ETL → PostgreSQL.

  Normalize inventory, sales, PO, and Safe Stock into a central database.

•Backoffice architecture

  Django reads from PostgreSQL and renders the backoffice UI.

  HTMX + Alpine.js add partial updates and lightweight UI state on top of server-side rendering.

  Designed so that Django buttons (e.g., “Sync now”, “Reload cache”) can trigger Flask automation APIs behind the scenes.

---

7. My Role

•Project Leadership (End-to-End): Led the project through the entire lifecycle, encompassing initial requirements gathering, architectural design, full-stack implementation, and integration with existing data sources.

•Business Analysis & Data Modeling:

Defined core operational KPIs and clarified how inventory, sales, POs, and Safe Stock should be visualized and unified.

Defined the required business entities and translated them into a normalized PostgreSQL schema and robust Django models, establishing the single source of truth for all commerce operations data.

•Full-Stack Implementation:

Developed the Django backend logic, views, and templates.

Implemented the admin interface using the HTMX + Alpine.js + Bootstrap stack to achieve a fast, quasi-SPA user experience without heavy JS framework overhead.

•Integration: Established the connection with the existing Shopify/Google Sheets → PostgreSQL data pipeline to ensure the backoffice always utilizes up-to-date, integrated operational data.

---

8.Further Reading 

• [ETL Sample Code](/docs/sample_code_ETL.md)

• [Shopify MetaField Update sample Code](/docs/sample_code_Shopify_MetaField.md)

• [Sample Screenshot](/docs/Screenshot_bigboard2.png)


---
