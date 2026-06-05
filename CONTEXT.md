# solar-sync

solar-sync automates a single household's EV charging so it draws preferentially from rooftop-solar excess rather than grid power. It replaces a human-in-the-loop process of eyeballing one app and toggling switches in another.

## Language

**EVSE**:
The Electric Vehicle Supply Equipment — the wall-mounted box that advertises available current to the car. Does not contain a charger. In this project it is specifically the Emporia Classic Level 2 unit.
_Avoid_: EV charger, charger, charging station, wall box

**Charger**:
The component inside the car that converts AC delivered by the EVSE into DC for the traction battery. solar-sync never talks to the Charger; it talks to the EVSE.
_Avoid_: onboard charger (use Charger), inverter
