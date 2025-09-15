package main

import (
	"crypto/sha256"
	"encoding/hex"
	"encoding/json"
	"fmt"
	"strconv"
	"strings"
	"time"
)

type Block struct {
	Index     int
	Timestamp string
	Data      string
	PrevHash  string
	Hash      string
	Nonce     int
}

func (b *Block) calculateHash() string {
	record := strconv.Itoa(b.Index) + b.Timestamp + b.Data + b.PrevHash + strconv.Itoa(b.Nonce)
	h := sha256.Sum256([]byte(record))
	return hex.EncodeToString(h[:])
}

func mineBlock(b *Block, difficulty int) {
	target := strings.Repeat("0", difficulty)
	for {
		b.Hash = b.calculateHash()
		if strings.HasPrefix(b.Hash, target) {
			return
		}
		b.Nonce++
	}
}

func createGenesis() Block {
	b := Block{Index: 0, Timestamp: time.Now().String(), Data: "Genesis", PrevHash: "0"}
	mineBlock(&b, 3)
	return b
}

func nextBlock(prev Block, data string, difficulty int) Block {
	b := Block{Index: prev.Index + 1, Timestamp: time.Now().String(), Data: data, PrevHash: prev.Hash}
	mineBlock(&b, difficulty)
	return b
}

func main() {
	chain := []Block{createGenesis()}
	fmt.Println("Genesis:", chain[0].Hash)
	// create couple blocks
	for i := 1; i <= 3; i++ {
		b := nextBlock(chain[len(chain)-1], fmt.Sprintf("Block data %d", i), 3)
		chain = append(chain, b)
		fmt.Printf("Mined block %d | hash %s | nonce %d\n", b.Index, b.Hash, b.Nonce)
	}
	// print chain as JSON
	out, _ := json.MarshalIndent(chain, "", "  ")
	fmt.Println(string(out))
}
