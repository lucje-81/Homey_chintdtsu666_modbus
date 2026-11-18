# Chint DTSU666 (Modbus) – Homey-app

Homey Pro-app die een **Chint DTSU666** driefasen-kWh-meter uitleest via een RS485-naar-netwerkconverter.
Gemaakt voor een meter in de voeding van een laadpaal, met laadsessies en een dagteller. De app leest alleen: er wordt nooit naar de meter geschreven.

## Verbinding

De DTSU666 praat Modbus RTU over RS485 (standaard 9600 baud, adres 1). De converter zet dat op je netwerk. Twee werkwijzen worden ondersteund:

| Converter-modus | Instelling in de app |
|---|---|
| Modbus TCP-gateway (bv. EW11 "Modbus TCP ⇔ RTU", Waveshare "Modbus TCP to RTU") | Modbus TCP |
| Transparant / "TCP server" zonder protocolconversie | Modbus RTU over TCP |

Laat je de instelling op **Automatisch**, dan probeert de app eerst Modbus TCP en daarna RTU over TCP, en onthoudt wat werkt.
Bij het toevoegen test de app de verbinding en toont de spanning op L1, zodat je meteen weet of alles klopt.

Let op: op een RS485-bus mag maar één toestel de meter uitlezen. Laat dus geen ander programma (bv. een test vanaf je pc) tegelijk pollen.

## Wat de app toont

| Waarde | Register (holding, FLOAT32) | Factor |
|---|---|---|
| Vermogen totaal / L1 / L2 / L3 | 0x2012 / 0x2014 / 0x2016 / 0x2018 | 0,1 W |
| Spanning L1 / L2 / L3 | 0x2006 / 0x2008 / 0x200A | 0,1 V |
| Stroom L1 / L2 / L3 | 0x200C / 0x200E / 0x2010 | 0,001 A |
| Arbeidsfactor (totaal) | 0x202A | 0,001 |
| Frequentie | 0x2044 | 0,01 Hz |
| Energie totaal afname | 0x101E | kWh |
| Energie totaal teruglevering | 0x1028 | kWh |

Daarnaast berekent de app:
- **Aan het laden** (ja/nee), **Energie laadsessie** (kWh) en **Duur laadsessie** (min)
- **Energie vandaag** (kWh, vanaf middernacht)

`Energie totaal (afname)` is de hoofdmeter voor Homey Energy.

### Laadsessies

- Een sessie start zodra het vermogen boven **Laden start boven** komt (standaard 1000 W).
- Ze stopt als het vermogen langer dan **Stopvertraging** (standaard 300 s) onder **Laden stopt onder** blijft (standaard 200 W). Zo telt een korte pauze van de auto niet als het einde.
- Na de sessie blijven energie en duur van de laatste sessie zichtbaar tot de volgende start.

Is de meter omgekeerd aangesloten (negatief vermogen tijdens het laden)? Zet dan **Richting omkeren** aan.

## Flowkaarten

- **Wanneer**
  - *Laden is gestart* (tag: vermogen)
  - *Laden is gestopt* (tags: energie kWh, duur min, meterstand begin/einde)
  - *Stroom op een fase steeg boven … A* (tags: fase, stroom), handig als bewaking van je aansluiting
  - *Waarde van [extra register] is veranderd*
- **En**
  - *Is / is niet aan het laden*
  - *Energie laadsessie is / is niet hoger dan … kWh*
  - *[extra register] is / is niet hoger dan …*
- Alle waarden zijn ook beschikbaar als tag en in Insights.

Voorbeeld: **Wanneer** *Laden is gestopt* → **Dan** melding "Auto geladen: [energie] kWh in [duur] min".

## Extra registers

Apparaatinstellingen → **Extra registers (alleen lezen)**, één per regel:

```
naam ; input|holding ; adres ; type ; factor ; eenheid
Blindvermogen ; holding ; 0x201A ; FLOAT32 ; 0.1 ; var
Spanning L1-L2 ; holding ; 0x2000 ; FLOAT32 ; 0.1 ; V
Arbeidsfactor L1 ; holding ; 0x202C ; FLOAT32 ; 0.001 ;
```

Adressen mogen decimaal of hexadecimaal (`0x…`). Types: FLOAT32, UINT16, INT16, UINT32, INT32.

## Installeren

```bash
npm install -g homey
cd chint-dtsu666
npm install
homey login
homey app install
```

Voeg daarna een apparaat toe: *DTSU666 energiemeter*. Vul IP-adres en poort van de converter in, plus het Modbus-adres van de meter, en klik op *Verbinding testen*.

## Bestanden

- `lib/modbus.ts`: Modbus-client (TCP en RTU over TCP, met CRC)
- `lib/dtsu666.ts`: registerkaart, uitlezen in blokken, protocolherkenning
- `lib/custom-registers.ts`: extra registers
- `drivers/dtsu666/device.ts`: polling, laadsessies, dagteller, flowtriggers
- `drivers/dtsu666/pair/start.html`: toevoegen met verbindingstest

Licentie: MIT.
