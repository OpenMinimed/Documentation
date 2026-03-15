

## Reconnect

- [ ] test reconnect on an esp32 @palmarci

## Documentation

- [ ] write proper SAKE docs @palmarci
- [ ] move com matrix from old repo @palmarci

## GATT

- CGM stuff
    - [X] current sg
    - [ ] calibration status
    - [ ] session start, session id
    - [ ] sensor remaining life time (total lifetime - session time?)
    - [ ] trend info
        - See characteristic _CGM Measurement_, field _CGM Trend Information_. We already parse that in our example app.
        - Pump's manual gives a relation between rise/fall rates and arrows displayed:
            - 1 arrow: SG has been rising or falling at a rate of 20-40 mg/dL over the last 20 minutes, or 1-2 mg/dL per minute.
            - 2 arrows: SG has been rising or falling at a rate of 40-60 mg/dL over the last 20 minutes, or 2-3 mg/dL per minute.
            - 3 arrows: SG has been rising or falling at a rate of more than 60 mg/dL over the last 20 minutes, or more than 3 mg/dL per minute.
    - [ ] time in range
        - characteristic _IDD Status Reader Control Point_ with custom Medtronic Op Code 0x401 (Get TIR Data) -> Response Op Code 0x402 (Get TIR Data Response)

- device info
  - [ ] pump name
    - characteristic _Serial Number String_ in standard _Device Information Service_
    - characteristic _Device Name_ in standard _Generic Access Service_ (for its BLE device name)
  - [ ] battery percentage
    - pump implements _Battery_ service which has _Battery Level_ characteristic
    - but there is also characteristic _GST Battery Level_ in the _Insulin Delivery Service_
  - [ ] version infos
    - characteristic _Hardware Revision String_ and _Firmware Revision String_ in standard _Device Information Service_
  - [ ] model number
    - characteristic _Model Number String_ in standard _Device Information Service_
  - [ ] used display format (mmol/l?)
    - characteristic _IDD Feature_, field _Flags_, bit _Glucose Unit mg/dL Used_

- pump status 
  - [ ] current date time
  - [ ] smartguard state
  - [ ] insulin amounts
    - [ ] currently active
    - [ ] left in the tank
    - [ ] active basal rate if not in smartguard

- history data
  - [ ] food intake
  - [ ] insulin intake
  - [ ] calibration (manual blood glucose enter)
  - [ ] suspends?
  - [ ] sport activity?

- annunciations
  - [ ] cgm (low, high, before low/high)
  - [ ] idd 
  - [ ] research commands to clear alarms and warnings