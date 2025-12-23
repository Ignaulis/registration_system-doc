# Registration System

## LT (trumpai)
Registracijų sistema su formų kūrimu ir vietų (capacity) kontrole. Administratorius gali kurti formas su dienomis (pirmadienis–sekmadienis) ir kiekvienai dienai pridėti neribotą veiklų skaičių (laikas + vietų skaičius). Kiekviena forma turi unikalų viešą URL (UUID), kurį galima dalintis. Dalyviai gali registruotis tik į veiklas, kuriose dar yra laisvų vietų. Po registracijos automatiškai siunčiamas patvirtinimo el. laiškas su pasirinkimų suvestine ir atšaukimo galimybe. Admin panelėje galima peržiūrėti, filtruoti pagal veiklą, redaguoti, trinti ir atšaukti registracijas. Admin prisijungimas per Google OAuth, prieiga valdoma `admins` lentele DB.

📄 Išsami dokumentacija (LT): ./doc_lt.md

---

## EN (short)
A registration system with a form builder and capacity control. Admins can create forms with multiple days (Monday–Sunday) and add any number of activities per day (time slot + seat limit). Each form has a unique public URL (UUID) that can be shared. Applicants can only select activities with available seats. After submission, a confirmation email is sent with a clear summary and a cancellation option. The admin panel supports viewing, filtering by activity, editing, deleting, and canceling registrations. Admin login uses Google OAuth, with access controlled via an `admins` allowlist table in the database.

📄 Full documentation (EN): ./doc_en.md
