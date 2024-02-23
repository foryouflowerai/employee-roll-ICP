# Employee Management Canister

The Employee Management Canister is a backend system designed to manage employee information, attendance records, payroll details, and analysis within an organization. It provides a set of functions for adding, retrieving, updating, and deleting employees, attendance records, and payroll details.

## Features

- **Employee Management**: Add, retrieve, update, and delete employee information including name, email, phone, address, department, designation, hire date, and salary.
- **Attendance Management**: Add, retrieve, and update attendance records for employees including check-in and optional check-out times.
- **Payroll Management**: Add, retrieve, and analyze payroll details including basic salary, allowances, and net salary for employees.
- **Search Functionality**: Search for employees by name to retrieve their information.
- **Comprehensive Analysis**: Analyze employee attendance to calculate total days, present days, and absent days.
- **Data Storage**: Utilizes `StableBTreeMap` data structures for efficient storage and retrieval of employee, attendance, and payroll records.

## Usage

### Installation

To use the Employee Management Canister in your project, you can install it via npm:

```bash
npm install azle
```

### Importing

You can import the necessary functions and data structures from the "azle" library:

```javascript
import { query, update, text, Record, StableBTreeMap, Variant, Vec, Ok, Err, Opt, Some, None, nat64, Result, Canister } from "azle";
// @ts-ignore
import { v4 as uuidv4 } from "uuid";
```

### Data Structures

The Employee Management Canister defines the following data structures:

- **Employee**: Represents an employee with properties such as id, name, email, phone, address, department, designation, hire date, and salary.
- **Attendance**: Represents an attendance record with properties id, employeeId, date, checkInTime, and checkOutTime.
- **Payroll**: Represents payroll details with properties id, employeeId, date, month, year, basicSalary, allowances, and netSalary.
- **Payslip**: Represents a payslip with properties id, employeeId, employeeName, date, month, year, basicSalary, allowances, and netSalary.
- **EmployeeAnalysis**: Represents the result of employee attendance analysis with properties employeeName, totalDays, presentDays, and absentDays.

### Functions

The Employee Management Canister provides the following functions:

#### Employee Management

- **addEmployee**: Add a new employee to the system.
- **getEmployees**: Retrieve all employees stored in the system.
- **getEmployee**: Retrieve a specific employee by their ID.
- **searchEmployee**: Search for employees by name.
- **updateEmployee**: Update an existing employee with new information.

#### Attendance Management

- **addAttendance**: Add an attendance record for an employee.
- **updateAttendance**: Update an attendance record with check-out time.
- **getAllAttendances**: Retrieve all attendance records stored in the system.
- **getAttendance**: Retrieve a specific attendance record by its ID.
- **getAttendanceByDate**: Retrieve attendance records for a specific date.
- **getEmployeeAttendances**: Retrieve attendance records for a specific employee.

#### Payroll Management

- **addPayroll**: Add payroll details for an employee.
- **getAllPayrolls**: Retrieve all payroll records stored in the system.
- **getPayroll**: Retrieve a specific payroll record by its ID.
- **getEmployeePayrolls**: Retrieve payroll records for a specific employee.
- **getYearPayrolls**: Retrieve payroll records for a specific year.
- **getMonthPayrolls**: Retrieve payroll records for a specific month.

### Error Handling

The system utilizes `Result` types to handle errors. Possible error types include NotFound, InvalidPayload, PaymentFailed, and PaymentCompleted.

### UUID Workaround

A workaround is provided to make the UUID package compatible with Azle. This enables the generation of unique identifiers for employees, attendance records, and payroll details.

## Contributing

Contributions to the Employee Management Canister are welcome! If you find any issues or have suggestions for improvements, please open an issue or submit a pull request on the GitHub repository.

## How to deploy canisters implemented in the course

### Ledger canister

`./deploy-local-ledger.sh` - deploys a local Ledger canister. IC works differently when run locally so there is no default network token available and you have to deploy it yourself. Remember that it's not a token like ERC-20 in Ethereum, it's a native token for ICP, just deployed separately.
This canister is described in the `dfx.json`:

```json
 "ledger_canister": {
   "type": "custom",
   "candid": "https://raw.githubusercontent.com/dfinity/ic/928caf66c35627efe407006230beee60ad38f090/rs/rosetta-api/icp_ledger/ledger.did",
   "wasm": "https://download.dfinity.systems/ic/928caf66c35627efe407006230beee60ad38f090/canisters/ledger-canister.wasm.gz",
   "remote": {
     "id": {
       "ic": "ryjl3-tyaaa-aaaaa-aaaba-cai"
     }
   }
 }
```

`remote.id.ic` - that is the principal of the Ledger canister and it will be available by this principal when you work with the ledger.

Also, in the scope of this script, a minter identity is created which can be used for minting tokens
for the testing purposes.
Additionally, the default identity is pre-populated with 1000_000_000_000 e8s which is equal to 10_000 * 10**8 ICP.
The decimals value for ICP is 10**8.

List identities:
`dfx identity list`

Switch to the minter identity:
`dfx identity use minter`

Transfer ICP:
`dfx ledger transfer <ADDRESS>  --memo 0 --icp 100 --fee 0`
where:

- `--memo` is some correlation id that can be set to identify some particular transactions (we use that in the marketplace canister).
- `--icp` is the transfer amount
- `--fee` is the transaction fee. In this case it's 0 because we make this transfer as the minter idenity thus this transaction is of type MINT, not TRANSFER.
- `<ADDRESS>` is the address of the recipient. To get the address from the principal, you can use the helper function from the marketplace canister - `getAddressFromPrincipal(principal: Principal)`, it can be called via the Candid UI.

### Internet identity canister

`dfx deploy internet_identity` - that is the canister that handles the authentication flow. Once it's deployed, the `js-agent` library will be talking to it to register identities. There is UI that acts as a wallet where you can select existing identities
or create a new one.

### Marketplace canister

`dfx deploy dfinity_js_backend` - deploys the marketplace canister where the business logic is implemented.
Basically, it implements functions like add, view, update, delete, and buy products + a set of helper functions.

Do not forget to run `dfx generate dfinity_js_backend` anytime you add/remove functions in the canister or when you change the signatures.
Otherwise, these changes won't be reflected in IDL's and won't work when called using the JS agent.

### Marketplace frontend canister

`dfx deploy dfinity_js_frontend` - deployes the frontend app for the `dfinity_js_backend` canister on IC.
