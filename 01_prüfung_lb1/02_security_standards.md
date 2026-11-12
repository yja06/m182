# 02 – Sicherheitsprinzipien, Richtlinien & Standards

## 1. Das Prinzip der geringsten Berechtigungen (Least Privilege)
- **Definition:** Benutzende, Prozesse und Programme erhalten immer nur exakt die minimal notwendigen Berechtigungen, die zwingend zur Erfüllung ihrer Aufgabe erforderlich sind.
- **Ziel:** Einschränkung des Schadensausmasses bei einer Kompromittierung (Blast Radius minimieren).
- **Umsetzung:**
  * Keine tägliche Arbeit mit Administrator-/Root-Rechten.
  * Zeitlich befristete Rechteerhöhung (z. B. via `sudo`, Just-In-Time Access).
  * Trennung von Benutzer- und Administratorkonten.

## 2. Sicherheitsstandards im Vergleich

### ISO/IEC 27001
- **Was ist es?** International anerkannte Norm für Informationssicherheits-Managementsysteme (ISMS).
- **Fokus:** Ganzheitlicher, organisatorischer und technischer Rahmen (Plan-Do-Check-Act-Zyklus).
- **Nutzen:** Zertifizierbarer Nachweis gegenüber Kunden und Partnern, dass Sicherheit systematisch gesteuert wird.

### IKT-Minimalstandard (Schweiz)
- **Was ist es?** Empfehlungen des Bundesamtes für wirtschaftliche Landesversorgung (BWL) / BACS für Schweizer Unternehmen.
- **Fokus:** Praktischer, pragmatischer Massnahmenkatalog zur Härtung kritischer Infrastrukturen und KMU.
- **Struktur:** Orientiert sich an den 5 Phasen: Identifizieren, Schützen, Erkennen, Reagieren, Wiederherstellen (NIST-basiert).

## 3. Sicherheitsrichtlinien (Security Policies)
- **Passwortrichtlinie:** Minimallänge (mind. 12–14 Zeichen), keine erzwungene häufige Rotation ohne Anlass, Einsatz von MFA/2FA, Prüfung gegen bekannte Leak-Listen.
- **Acceptable Use Policy (AUP):** Regelt, wie Firmengeräte, Internet und E-Mail genutzt werden dürfen (z. B. Privatnutzung, Download von Software).
- **Incident Response Plan:** Definierter Ablauf bei Sicherheitsvorfällen (Wer muss wen wann informieren?).
