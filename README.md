import matplotlib.pyplot as plt
import pandas as pd

# ---------------------------------------------------------
# 1. Excel-Tabelle einlesen (Ertragsdaten)
# ---------------------------------------------------------
# Wir überspringen die ersten 12 Zeilen, da diese Metadaten enthalten.
# Wir lesen 26 Zeilen ein (Zeitraum 2000-2025).
ertrag_excel = pd.read_excel("Tabelle Grünland Ertrag 2000-2025.xlsx", skiprows=12, nrows=26, header=None)

# Die Jahreszahlen befinden sich in der zweiten Spalte (Index 1)
jahr = ertrag_excel[1].astype(int)

# Bindestriche durch 0 ersetzen für die 3 Schnitte: Heu, Gruimat und Pofl
# Dies ist notwendig, damit wir mathematische Berechnungen durchführen können.
wiesenzweimähdig = pd.to_numeric(ertrag_excel[5].astype(str).str.replace("-", "0"))
wiesendreimähdig = pd.to_numeric(ertrag_excel[6].astype(str).str.replace("-", "0"))
streuwiesen = pd.to_numeric(ertrag_excel[7].astype(str).str.replace("-", "0"))

# Alles plus rechnen für den Gesamtertrag
gesamt_ertrag = wiesenzweimähdig + wiesendreimähdig + streuwiesen

# ---------------------------------------------------------
# 2. Wetter-CSV einlesen
# ---------------------------------------------------------
wetter_csv = pd.read_csv("Messstationen Jahreswetterdaten.csv")

# Das Jahr aus dem langen Datumstext extrahieren (die ersten 4 Zeichen der 'time' Spalte)
wetter_csv["Jahr"] = wetter_csv["time"].str[:4].astype(int)

# Den durchschnittlichen Regen pro Jahr ausrechnen (Aggregation der Messstationen)
regen_pro_jahr = wetter_csv.groupby("Jahr")["rr"].mean().reset_index()

# ---------------------------------------------------------
# 3. Zusammenführung der Daten (Merge)
# ---------------------------------------------------------
# Wir erstellen einen DataFrame für den Ertrag und verbinden ihn mit den Wetterdaten
tabelle_ertrag = pd.DataFrame({"Jahr": jahr, "Ertrag": gesamt_ertrag})
daten_fertig = pd.merge(tabelle_ertrag, regen_pro_jahr, on="Jahr")

# ---------------------------------------------------------
# 4. Visualisierung (Diagramm erstellen)
# ---------------------------------------------------------
fig, ax1 = plt.subplots(figsize=(10, 6))

# Grüne Balken für den Ertrag (linke Y-Achse)
ax1.bar(daten_fertig["Jahr"], daten_fertig["Ertrag"], color="green", alpha=0.5, label="Ertrag (t)")
ax1.set_xlabel("Jahr")
ax1.set_ylabel("Ertrag (Tonnen)", color="green")
ax1.tick_params(axis='y', labelcolor='green')

# Zweite Achse rüberspiegeln für die blaue Regen-Linie (rechte Y-Achse)
ax2 = ax1.twinx()
ax2.plot(daten_fertig["Jahr"], daten_fertig["rr"], color="blue", marker="o", label="Niederschlag (mm)")
ax2.set_ylabel("Niederschlag (mm)", color="blue")
ax2.tick_params(axis='y', labelcolor='blue')

# Titel und Layout
plt.title("Tirol: Korrelation Ertrag vs. Niederschlag (2000-2025)")a
fig.tight_layout()

# Diagramm anzeigen
plt.show()
