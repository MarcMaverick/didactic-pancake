// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract GreenLungToken is ERC20, Ownable {
    uint256 public constant MAX_SUPPLY = 1_000_000_000 * 10**18;  // 1 Milliarde GLT
    uint256 public totalCommunityContribution; // Gesamte eingezahlte 10% Community-Beiträge

    struct Member {
        uint256 votes; 
        bool isBoardMember;
        bool hasContributed; // Prüft, ob das Mitglied seinen 10% Community-Beitrag geleistet hat
    }

    mapping(address => Member) public members;
    address[] public boardMembers;
    
    // Adressen für verschiedene Zwecke
    address public projectReserve;
    address public partnershipReserve;
    address public teamReserve;
    address public communityReserve;

    // Slogan: Revolution - Ein Token allein ist schwach, aber zusammen sind sie sehr stark.
    constructor(
        address _projectReserve,
        address _partnershipReserve,
        address _teamReserve,
        address _communityReserve
    ) ERC20("Green Lung Token", "GLT") {
        projectReserve = _projectReserve;
        partnershipReserve = _partnershipReserve;
        teamReserve = _teamReserve;
        communityReserve = _communityReserve;

        // Zuweisung der Token-Menge auf die entsprechenden Reserven
        _mint(projectReserve, 600_000_000 * 10**18); // 600 Millionen GLT
        _mint(partnershipReserve, 100_000_000 * 10**18); // 100 Millionen GLT
        _mint(teamReserve, 100_000_000 * 10**18); // 100 Millionen GLT
        _mint(communityReserve, 200_000_000 * 10**18); // 200 Millionen GLT
    }

    // Funktion, um Mitglieder hinzuzufügen und sicherzustellen, dass sie im Vorstand sind und 3 Stimmen erhalten
    function addBoardMember(address memberAddress) external onlyOwner {
        require(!members[memberAddress].isBoardMember, "Already a board member");
        members[memberAddress] = Member({ votes: 3, isBoardMember: true, hasContributed: false });
        boardMembers.push(memberAddress);
    }

    // Funktion, um andere ökologische Tokens in GLT zu vereinen und den Wert von GLT basierend auf diesen Tokens festzulegen
    function uniteTokens(address[] calldata tokenAddresses, uint256[] calldata amounts, uint256[] calldata tokenValues) external onlyOwner {
        require(tokenAddresses.length == amounts.length && amounts.length == tokenValues.length, "Mismatched arrays");

        uint256 totalValue;
        for (uint256 i = 0; i < tokenAddresses.length; i++) {
            ERC20 token = ERC20(tokenAddresses[i]);
            require(token.transferFrom(msg.sender, address(this), amounts[i]), "Token transfer failed");
            totalValue += amounts[i] * tokenValues[i]; // Summiere den Gesamtwert der vereinheitlichten Tokens
        }

        // Mint GLT im Wert des vereinheitlichten Tokens
        _mint(msg.sender, totalValue);
    }

    // Funktion zur Berechnung und Einzahlung von 10% in die Community
    function contributeToCommunity() external {
        require(members[msg.sender].isBoardMember, "Not a board member");
        require(!members[msg.sender].hasContributed, "Already contributed");
        
        uint256 tenPercent = balanceOf(msg.sender) / 10;
        require(balanceOf(msg.sender) >= tenPercent, "Insufficient balance for contribution");
        
        _transfer(msg.sender, communityReserve, tenPercent);
        members[msg.sender].hasContributed = true;
        totalCommunityContribution += tenPercent;
    }

    // Abstimmungsfunktion für Board-Mitglieder, um Projekte zu unterstützen
    function voteOnProject(uint256 votes) external {
        require(members[msg.sender].isBoardMember, "Not a board member");
        require(members[msg.sender].votes >= votes, "Not enough votes");

        members[msg.sender].votes -= votes;
        // Weitere Logik zur Projektunterstützung kann hier hinzugefügt werden
    }

    // Funktion, um die Adressen für verschiedene Reserves zu ändern
    function updateReserves(
        address _projectReserve,
        address _partnershipReserve,
        address _teamReserve,
        address _communityReserve
    ) external onlyOwner {
        projectReserve = _projectReserve;
        partnershipReserve = _partnershipReserve;
        teamReserve = _teamReserve;
        communityReserve = _communityReserve;
    }
}
