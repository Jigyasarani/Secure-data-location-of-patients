# Secure-data-location-of-patients
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract DataLocationRegistry {
    address public owner;
    uint256 public dataCount;

    struct DataRecord {
        address owner;
        string dataHash;
        string location;
        uint256 timestamp;
    }

    mapping(uint256 => DataRecord) public dataRecords;

    event DataRecordAdded(uint256 indexed id, address indexed owner, string dataHash, string location, uint256 timestamp);

    constructor() {
        owner = msg.sender;
        dataCount = 0;
    }

    modifier onlyOwner() {
        require(msg.sender == owner, "Only the contract owner can perform this action");
        _;
    }

    function addDataRecord(string memory _dataHash, string memory _location) public {
        require(bytes(_dataHash).length > 0, "Data hash cannot be empty");
        require(bytes(_location).length > 0, "Location cannot be empty");

        dataCount++;
        dataRecords[dataCount] = DataRecord({
            owner: msg.sender,
            dataHash: _dataHash,
            location: _location,
            timestamp: block.timestamp
        });

        emit DataRecordAdded(dataCount, msg.sender, _dataHash, _location, block.timestamp);
    }

    function verifyDataLocation(uint256 _id, string memory _dataHash, string memory _location) public view returns (bool) {
        require(_id > 0 && _id <= dataCount, "Invalid data ID");

        DataRecord storage record = dataRecords[_id];
        return (keccak256(abi.encodePacked(_dataHash, _location)) == keccak256(abi.encodePacked(record.dataHash, record.location)));
    }
}
