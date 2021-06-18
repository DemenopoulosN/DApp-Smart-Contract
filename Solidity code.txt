// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;


contract vaccineContract {
    
    // These will be assigned at the construction
    // phase, where `msg.sender` is the account
    // creating this contract.
    address public owner = msg.sender;
    
    uint[2] dosesAvailable = [1000, 1000];
    string[2] vaccineType = ["mRNA", "viralVector"];
    string[2] hospitals = ["St Thomas' Hospital", "Hammersmith Hospital"];
    string vacFinal = "";
    string locFinal = "";
    uint dayFinal_1st = 0;
    uint dayFinal_2nd = 0;
    uint monthFinal_1st = 0;
    uint monthFinal_2nd = 0;

    struct Citizen { 
        uint afm;
        string name;
        string surname;
        uint dosesAdministered;
        bool vaccineChosen;
        bool datetimeChosen;
        bool hasAppointment;
        bool vaccinated;
        string appointmentID;
    }
    mapping (uint => Citizen) Citizens;
    
    
    struct Appointment { 
        string vaccineType;
        string location;
        string date1;
        string date2;
    }
    mapping (string => Appointment) Appointments;
    
    
    
    // If this modifier is used, it will
    // prepend a check that only passes
    // if the function is called from
    // a certain address.
    modifier onlyBy(address _account)
    {
        require(
            msg.sender == _account,
            "Sender not authorized."
        );
        _;
    }
        
        
    //
    //citizen 
    //
    function SetCitizenInfo(uint amka, uint afm, string memory name, string memory surname) public{
        require(bytes(uintToString(amka)).length == 11, "Wrong input! AMKA must be 11 charachters long!");
        require(bytes(uintToString(afm)).length == 9, "Wrong input! AFM must be 9 charachters long!");
        Citizens[amka].afm = afm;
        Citizens[amka].name = name;
        Citizens[amka].surname = surname;
        
        Citizens[amka].dosesAdministered = 0;
        Citizens[amka].appointmentID = "No appointment scheduled";
        Citizens[amka].vaccineChosen = false;
        Citizens[amka].datetimeChosen = false;
        Citizens[amka].hasAppointment = false;
        Citizens[amka].vaccinated = false;
    }
    
    
    function GetCitizenInfo(uint amka) public view returns(uint,string memory,string memory,bool,bool,string memory){
        return (Citizens[amka].afm,Citizens[amka].name,Citizens[amka].surname,Citizens[amka].hasAppointment,Citizens[amka].vaccinated,Citizens[amka].appointmentID);
    }
    
    
    //
    //choose vaccine
    //
    function chooseVaccineType(uint amka, string memory vType) public{
        require(!Citizens[amka].vaccinated, "You have already been vaccinated!");
        require(keccak256(abi.encodePacked(vType)) == keccak256(abi.encodePacked("mRNA")) || keccak256(abi.encodePacked(vType)) == keccak256(abi.encodePacked("viralVector")), "Wrong input! You can choose between 'mRNA' and 'viralVector' vaccines.");
            if(Citizens[amka].hasAppointment == false){
                for (uint i = 0; i < vaccineType.length; i++) {
                    if(keccak256(abi.encodePacked(vaccineType[i])) == keccak256(abi.encodePacked(vType))){
                        if(dosesAvailable[i] >= 2){
                            dosesAvailable[i] = dosesAvailable[i] - 2 ;
                            Citizens[amka].vaccineChosen = true;
                            vacFinal = vType;
                        }
                        else{
                            uint index = (~i+1) + 1;
                            require(!(dosesAvailable[index] >= 2), "Vacccine type chosen not available! Choose another vaccine type!");
                            require(dosesAvailable[index] >= 2, "No vacccines available at this time!");
                        }
                    }
                }
            }
            else{
                //wants to change vaccine type
                if(keccak256(abi.encodePacked(Appointments[Citizens[amka].appointmentID].vaccineType)) != keccak256(abi.encodePacked(vType))){
                    for (uint i = 0; i < vaccineType.length; i++) {
                        if(keccak256(abi.encodePacked(vaccineType[i])) == keccak256(abi.encodePacked(vType))){
                            uint index = (~i+1) + 1;
                            if(dosesAvailable[i] >= 2){
                                dosesAvailable[i] = dosesAvailable[i] - 2 ;
                                dosesAvailable[index] = dosesAvailable[index] + 2 ;
                                vacFinal = vType;
                            }
                            else{
                                require(false, "This vaccine is not available. You can only stick with your currently selected vaccine.");
                            }
                        }
                    }
                }
            }
    }
    
    
    //
    // choose date for first dose vaccination
    //
    function chooseDate(uint amka, uint day, uint month) public{
        require(!Citizens[amka].vaccinated, "You have already been vaccinated!");
        require(!Citizens[amka].hasAppointment, "You already have an appointment!");
        require(Citizens[amka].vaccineChosen, "You have to choose a vaccine first!");
        require(day >= 1 && day <= 30,"Wrong input! Day must be between 1 and 30!");
        require(month >= 1 && month <= 12,"Wrong input! Month must be between 1 and 12!");
                dayFinal_1st = day;
                monthFinal_1st = month;
                if(keccak256(abi.encodePacked(vacFinal)) == keccak256(abi.encodePacked("mRNA"))){
                    if(day + 21 > 30){
                        dayFinal_2nd = day - 9;
                        if(month == 12){
                            monthFinal_2nd = 1;
                        }
                        else{
                            monthFinal_2nd = month + 1;
                        }
                    }
                    else{
                        dayFinal_2nd = day + 21;
                        monthFinal_2nd = month;
                    }
                }
                else{
                    dayFinal_2nd = day;
                    if(month + 2 > 12){
                        monthFinal_2nd = month - 10;
                    }
                    else{
                        monthFinal_2nd = month + 2;
                    }
                }
                Citizens[amka].datetimeChosen = true;
    }
    
    //
    //appointment
    //
    function makeAppointment(uint amka) public{
        require(!Citizens[amka].vaccinated, "You have already been vaccinated!");
        require(!Citizens[amka].hasAppointment, "You already have an appointment!");
        require(Citizens[amka].datetimeChosen, "You have to choose a date first!");
            string memory x = uintToString(rand());
            Citizens[amka].appointmentID = append(uintToString(amka), x);
            Appointments[Citizens[amka].appointmentID].vaccineType = vacFinal;
            Appointments[Citizens[amka].appointmentID].date1 = append(uintToString(dayFinal_1st), "/", uintToString(monthFinal_1st));
            Appointments[Citizens[amka].appointmentID].date2 = append(uintToString(dayFinal_2nd), "/", uintToString(monthFinal_2nd));
            for (uint i = 0; i < vaccineType.length; i++) {
                if(keccak256(abi.encodePacked(vaccineType[i])) == keccak256(abi.encodePacked(vacFinal))) {
                    Appointments[Citizens[amka].appointmentID].location = hospitals[i];
                }
            }
            Citizens[amka].hasAppointment = true;
    }
    
    
    function GetAppointmentInfo(string memory appointmentID) public view returns(string memory,string memory,string memory,string memory){
        return (Appointments[appointmentID].vaccineType,Appointments[appointmentID].location,Appointments[appointmentID].date1,Appointments[appointmentID].date2);
    }
    
    
    //
    //vaccination
    //
    function vaccinationUpdate(uint amka, string memory id) public onlyBy(owner) {
        require(keccak256(abi.encodePacked(Citizens[amka].appointmentID)) == keccak256(abi.encodePacked(id)), "Wrong appointment id input!");
            if(Citizens[amka].dosesAdministered == 0){
                Citizens[amka].dosesAdministered = 1;
            }
            else{
                Citizens[amka].dosesAdministered = 2;
                Citizens[amka].vaccinated = true;
            }
    }
    
    
    //
    //extra functions
    //
    function uintToString(uint _i) internal pure returns (string memory _uintAsString) {
        if (_i == 0) {
            return "0";
        }
        uint j = _i;
        uint len;
        while (j != 0) {
            len++;
            j /= 10;
        }
        bytes memory bstr = new bytes(len);
        uint k = len;
        while (_i != 0) {
            k = k-1;
            uint8 temp = (48 + uint8(_i - _i / 10 * 10));
            bytes1 b1 = bytes1(temp);
            bstr[k] = b1;
            _i /= 10;
        }
        return string(bstr);
    }
    
    function rand() public view returns(uint){
        uint seed = uint(keccak256(abi.encodePacked(
            block.timestamp + block.difficulty +
            ((uint(keccak256(abi.encodePacked(block.coinbase)))) / (block.timestamp)) +
            block.gaslimit + 
            ((uint(keccak256(abi.encodePacked(msg.sender)))) / (block.timestamp)) +
            block.number
        )));
    
        return (seed - ((seed / 1000) * 1000));
    }
    
    
    function append(string memory a, string memory b) internal pure returns (string memory) {
        return string(abi.encodePacked(a, b));
    }
    
    function append(string memory a, string memory b, string memory c) internal pure returns (string memory) {
        return string(abi.encodePacked(a, b, c));
    }
    
}