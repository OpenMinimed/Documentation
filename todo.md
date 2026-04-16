
## Documentation

- [ ] write proper SAKE docs @palmarci
- [ ] move com matrix from old repo @palmarci

## GATT

- CGM stuff
    - [X] current sg
    - [ ] calibration status
    - [x] session start, session id
	    - characterstic _CGM Session Start Time_
	    - characteristic _CGM Specific Ops Control Point_ with custom Medtronic Op Code 0x8c (Read Current Session ID) -> Response Op Code 0x8d (Read Current Session ID Response)
		    - this does not seem to be supported by the 780G, but session IDs are not needed as there only ever seems to be exactly one session which is started automatically
    - [x] sensor remaining life time
	    - characterstics _CGM Session Start Time_ and _CGM Session Run Time_
	    - compute the absolute end time from them, then compute the remaining time from current time to that end time
    - [X] trend info
        - See characteristic _CGM Measurement_, field _CGM Trend Information_. We already parse that in our example app.
        - Pump's manual gives a relation between rise/fall rates and arrows displayed:
            - 1 arrow: SG has been rising or falling at a rate of 20-40 mg/dL over the last 20 minutes, or 1-2 mg/dL per minute.
            - 2 arrows: SG has been rising or falling at a rate of 40-60 mg/dL over the last 20 minutes, or 2-3 mg/dL per minute.
            - 3 arrows: SG has been rising or falling at a rate of more than 60 mg/dL over the last 20 minutes, or more than 3 mg/dL per minute.
    - [X] time in range
        - characteristic _IDD Status Reader Control Point_ with custom Medtronic Op Code 0x401 (Get TIR Data) -> Response Op Code 0x402 (Get TIR Data Response)

- device info
  - [x] pump name
    - characteristic _Serial Number String_ in standard _Device Information Service_
    - characteristic _Device Name_ in standard _Generic Access Service_ (for its BLE device name)
  - [x] battery percentage
    - pump implements _Battery_ service which has _Battery Level_ characteristic
    - but there is also characteristic _GST Battery Level_ in the _Insulin Delivery Service_ for the transmitter
  - [x] version infos
    - characteristic _Hardware Revision String_ and _Firmware Revision String_ in standard _Device Information Service_
  - [x] model number
    - characteristic _Model Number String_ in standard _Device Information Service_
  - [ ] used display format (mmol/l?)
    - characteristic _IDD Feature_, field _Flags_, bit _Glucose Unit mg/dL Used_

- pump status 
  - [ ] current date time
  - [ ] smartguard state
    - characteristic _IDD Status Reader Control Point_ with custom Medtronic Op Code 0x3fd (Get Therapy Algorithm States) -> Response Op Code 0x3fe (Get Therapy Algorithm States Response)
  - insulin amounts
    - [X] currently active
      - characteristic _IDD Status Reader Control Point_ with standard Op Code 0x3f3 (Get Insulin On Board) -> Response Op Code 0x3fc (Get Insulin On Board Response)
    - [X] left in the tank
      - characteristic _IDD Status_, field _Reservoir Remaining Amount_
      - [ ] expected wear out time in hours (based on 2 days avg of used insulin)
    - [ ] active basal rate (if not in smartguard?)
      - characteristic _IDD Status Reader Control Point_ with standard Op Code 0x365 (Get Active Basal Rate Delivery) -> Response Op Code 0x36a (Get Active Basal Rate Delivery Response)

- history data
  - [x] food intake
	- event type 0xf005 (Meal)
  - [x] insulin intake
    - event types 0x005a (Bolus Programmed Part 1 of 2), 0x0066 (Bolus Programmed Part 2 of 2), 0x0069 (Bolus Delivered Part 1 of 2), 0x0096 (Bolus Delivered Part 2 of 2)
  - [x] calibration (manual blood glucose enter)
    - event types 0xf007 (BG Reading), 0xf008 (Calibration Complete), 0xf009 (Calibration Rejected)
  - [ ] suspends?
  - [ ] sport activity?

- annunciations
  - [ ] cgm (low, high, before low/high)
  - [ ] idd 
  - [ ] research commands to clear alarms and warnings
