# DE-OPERATOR

Reference notes on the regulatory perimeter for a German-resident
solo Storj SNO. Written in English; German statutory terms and form
names appear verbatim because their legal weight is in the original
wording.

> **Wichtiger Hinweis / Important notice:** This document is not tax
> or legal advice. It is a planning-level summary of the regulatory
> perimeter for a DE-resident solo operator. Before filing anything,
> opening a Gewerbe, or making representations to the Finanzamt,
> consult a Steuerberater (tax adviser) and, where the activity
> approaches custodial storage for third parties, a Rechtsanwalt
> (attorney) familiar with BaFin licensing perimeters.

## What this document is

- A planning checklist for a DE-resident Einzelunternehmer
  (sole proprietor) operating one to ten Storj nodes from a German
  address
- A pointer to the relevant statutes and form names so an operator can
  read them in the original
- A pre-conversation document to bring to a Steuerberater

## What this document is not

- Steuerberatung in the meaning of §3 StBerG (tax-advice law)
- A substitute for the actual statutes; statutory text is quoted in
  German with English summary beneath
- Legal advice on cross-border operation (operators outside DE are
  outside the scope of this reference)
- Tax planning for the corporate forms (UG, GmbH, GbR); Einzelunternehmer
  is the assumed form

---

## 1. Gewerbeanmeldung (trade registration)

Operating a Storj node for ongoing income meets the definition of a
Gewerbe (commercial activity) under §14 GewO. The activity is
ongoing, profit-intended, and outward-facing, the three elements
the Gewerbeordnung requires.

> **§14 GewO (Gewerbeordnung), excerpt:** "Wer den selbständigen
> Betrieb eines stehenden Gewerbes [...] anfängt, muss dies der
> zuständigen Behörde gleichzeitig anzeigen."

Translation summary: independent commercial activity must be notified
to the responsible authority (the local Gewerbeamt) at the same time
it begins.

**Practical step:** file the Gewerbeanmeldung at the responsible
Gewerbeamt (usually the city or municipality where the operator is
resident). Filing fee is typically 20 to 60 Euro depending on the
city.

Form: **Gewerbeanmeldung** (sometimes digital via the Gewerbeamt's
portal). The form requires:

- Operator name and address
- Description of the activity, for Storj, a description such as
  "Betrieb dezentraler Speicherknoten im Netzwerk Storj"
  (operation of decentralised storage nodes in the Storj network)
- Start date
- Whether the activity is haupt- or nebenberuflich
  (main or secondary occupation)

After filing the Gewerbeamt forwards the notification to the
Finanzamt automatically. The Finanzamt then sends the
**Fragebogen zur steuerlichen Erfassung** (questionnaire for tax
registration), which the operator must complete to receive a
Steuernummer (tax number) for the Gewerbe.

## 2. §22 EStG (Sonstige Einkünfte) vs Gewerbeeinkünfte: classification

The income tax treatment depends on the classification of the
activity. Two surfaces are in play.

### Gewerbeeinkünfte (§15 EStG, commercial income)

Once Gewerbeanmeldung is filed, the income is classified as
Einkünfte aus Gewerbebetrieb (commercial income) under §15 EStG and
not as Sonstige Einkünfte. This is the dominant case for an operator
filing a Gewerbe.

> **§15 Abs. 1 EStG, excerpt:** "Einkünfte aus Gewerbebetrieb sind
> Einkünfte aus gewerblichen Unternehmen."

Filing surface: **Anlage G** (commercial income annex) on the
Einkommensteuererklärung (income tax return).

### Sonstige Einkünfte (§22 Nr. 3 EStG, miscellaneous income)

If the operator does not file a Gewerbe and the activity is occasional
and small-scale, income may instead be classified as Sonstige
Einkünfte under §22 Nr. 3 EStG.

> **§22 Nr. 3 EStG, excerpt:** "Sonstige Einkünfte sind [...]
> Einkünfte aus Leistungen, [...] soweit sie weder zu anderen
> Einkunftsarten [...] gehören."

There is a Freigrenze (allowance threshold) of **256 Euro per
calendar year** under §22 Nr. 3 Satz 2 EStG. Below the Freigrenze the
income is tax-free; above it the entire amount is taxable, not only
the portion above the threshold.

> **§22 Nr. 3 Satz 2 EStG, excerpt:** "Einkünfte dieser Art sind
> nicht einkommensteuerpflichtig, wenn sie weniger als 256 Euro im
> Kalenderjahr betragen haben."

Filing surface: **Anlage SO** (sonstiges Einkommen annex).

**Practical reading for a Storj operator:** running multiple nodes
with intent to grow is commercial activity and belongs under §15
EStG with a Gewerbeanmeldung. A single test node generating under
256 Euro per year could be argued under §22 Nr. 3 EStG with no
Gewerbeanmeldung, but the operator carries the risk of the Finanzamt
disagreeing and reclassifying retroactively. A Steuerberater is the
correct surface for this judgement.

## 3. Gewerbesteuer (trade tax) and the §11 GewStG Freibetrag

Once classified as Gewerbe, the income is subject to Gewerbesteuer
(trade tax). The relevant Freibetrag (tax-free amount) for natural
persons and Personengesellschaften (partnerships) is **24,500 Euro
per year** under §11 Abs. 1 Nr. 1 GewStG.

> **§11 Abs. 1 Nr. 1 GewStG, excerpt:** "Der Gewerbeertrag wird auf
> volle hundert Euro nach unten abgerundet und [...] um einen
> Freibetrag in Höhe von 24,500 Euro [...] gekürzt, höchstens jedoch
> in Höhe des abgerundeten Gewerbeertrags."

Below 24,500 Euro Gewerbeertrag (commercial profit) the
Gewerbesteuer is zero. A solo Storj operator with one to ten nodes
operating below this threshold owes no Gewerbesteuer, but the
declaration (Gewerbesteuererklärung) is still required.

Filing surface: **Gewerbesteuererklärung** (separate from the
Einkommensteuererklärung).

## 4. Umsatzsteuer (VAT) and the Kleinunternehmerregelung

Storj rewards are paid in STORJ token (an ERC-20 token). The
Finanzamt treats the receipt as a sonstige Leistung (other service)
for Umsatzsteuer purposes. Whether VAT applies depends on the
operator's status:

### Kleinunternehmerregelung (§19 UStG)

If turnover is below **22,000 Euro in the previous calendar year and
expected below 50,000 Euro in the current** under §19 Abs. 1 UStG,
the operator can opt into the Kleinunternehmerregelung (small-business
rule) and not charge VAT on outgoing invoices and not deduct input
VAT on purchases.

> **§19 Abs. 1 UStG, excerpt:** "Die für Umsätze [...] geschuldete
> Umsatzsteuer wird von Unternehmern, die im Inland [...] ansässig
> sind, nicht erhoben, wenn der Umsatz [...] im vorangegangenen
> Kalenderjahr 22,000 Euro nicht überstiegen hat und im laufenden
> Kalenderjahr 50,000 Euro voraussichtlich nicht übersteigen wird."

For a solo Storj operator below these thresholds (the case for any
realistic one-to-ten-node setup at current rates) the
Kleinunternehmerregelung is the default choice. Opting out is
possible but locks the operator into normal Umsatzsteuer obligations
for five years.

Filing surface: indicated on the Fragebogen zur steuerlichen
Erfassung when the Gewerbe is registered, or via a later written
notification to the Finanzamt.

### B2B reverse-charge to Storj Labs (out of scope here)

Storj Labs is a US entity. If an operator is not under the
Kleinunternehmerregelung, the supply to Storj Labs is a B2B service
subject to reverse-charge rules. This is out of scope for this
reference; consult a Steuerberater if Kleinunternehmer status no
longer applies.

## 5. Vetting period reality

A new Storj node enters a **vetting phase** (Vetting-Phase) before it
receives full traffic. During vetting the node receives less than
five percent of the traffic of a vetted node in the same satellite.

Expected duration: roughly 30 days per satellite under typical
conditions, sometimes longer if uptime is imperfect. The node must
be online and complete a sufficient volume of audit requests before
the satellite considers it vetted.

Practical implication: the first month produces near-zero income.
Plan accordingly. Buying hardware on the expectation of immediate
payback is a common mistake; the realistic horizon for break-even
on consumer hardware at current rates is measured in months, not
weeks. Operators run nodes for the architecture practice, the
hardware utilisation, or the conviction in the protocol, not for
fast cashflow.

## 6. Reasonable bookkeeping practice (advisory)

For a Storj Gewerbe with one to ten nodes:

- **Separate bank account.** Receipts from any rewards conversion go
  here. Hardware and electricity expenses are paid from here.
  Co-mingling with private accounts is the most common audit trigger.
- **Monthly Einnahmen-Überschuss-Rechnung (EÜR).** §4 Abs. 3 EStG
  permits the EÜR (cash-basis P&L) for operators below the §141 AO
  bookkeeping threshold (currently 600,000 Euro turnover or 60,000
  Euro profit per year). This is the small-operator default and is
  filed as **Anlage EÜR** with the Einkommensteuererklärung.
- **Track every STORJ-to-EUR conversion** with date, USD-EUR rate
  used, EUR equivalent. The Finanzamt taxes EUR-equivalent receipts
  at the time of conversion.
- **Track electricity cost separately.** Allocate by node-hour if
  multiple devices share an electricity bill; pure-Storj electricity
  is fully deductible as Betriebsausgabe.
- **Depreciate hardware.** AfA-Tabelle "AV" assigns servers and
  storage devices a Nutzungsdauer of three to five years. Items
  under 800 Euro net (Geringwertige Wirtschaftsgüter, §6 Abs. 2
  EStG) can be fully expensed in the year of acquisition.

## 7. What to bring to the Steuerberater

When the first Steuerberater conversation happens, the operator
should arrive with:

- A description of the activity (a Storj node operation summary)
- An estimate of node count, hardware budget, monthly electricity
  cost, expected monthly STORJ rewards
- A choice or pre-decision on Einzelunternehmer vs UG vs GmbH
  (Einzelunternehmer is the default for a solo SNO)
- A choice on Kleinunternehmerregelung
- Knowledge of the AfA-Tabelle treatment of the hardware

This document plus the operator's actual numbers is the working set.

---

## Statutory references (further reading, in German)

- §14 GewO: https://www.gesetze-im-internet.de/gewo/__14.html
- §15 EStG (Einkünfte aus Gewerbebetrieb): https://www.gesetze-im-internet.de/estg/__15.html
- §22 EStG (Sonstige Einkünfte): https://www.gesetze-im-internet.de/estg/__22.html
- §11 GewStG (Freibetrag): https://www.gesetze-im-internet.de/gewstg/__11.html
- §19 UStG (Kleinunternehmerregelung): https://www.gesetze-im-internet.de/ustg_1980/__19.html
- §4 Abs. 3 EStG (Einnahmen-Überschuss-Rechnung): https://www.gesetze-im-internet.de/estg/__4.html
- AfA-Tabelle AV: https://www.bundesfinanzministerium.de/Content/DE/Standardartikel/Themen/Steuern/Weitere_Steuerthemen/Betriebspruefung/AfA-Tabellen/afa-tabelle-AV.html

## Operational references

- Storj Node Operator documentation: https://docs.storj.io/node
- Storj community forum: https://forum.storj.io/
- Storj vetting period explainer: https://storj.dev/dcs/concepts/satellites/vetting

---

**Reminder:** Nothing in this document substitutes for a Steuerberater.
The DE tax surface changes; statutes are amended; the Finanzamt's
internal guidance is not always public. Consult.
