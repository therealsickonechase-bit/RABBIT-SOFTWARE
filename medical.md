# Bitcoin CLI Alias (Permanent)
Set-Alias bitcoin-cli '"C:\Program Files\Bitcoin\daemon\bitcoin-cli.exe"'

# Functions for Medical Block Hunting
function Get-BlockHeight { (bitcoin-cli getblockchaininfo | ConvertFrom-Json).blocks }
function Get-LatestBlock { bitcoin-cli getbestblockhash }
function Scan-MedicalBlock($height) {
    $hash = bitcoin-cli getblockhash $height
    $block = bitcoin-cli getblock $hash 1 | ConvertFrom-Json
    Write-Host "Block $height ($hash`:~$($block.nTx) tx)"
    
    # Check first 10 tx for OP_RETURN medical data
    0..9 | ForEach-Object {
        $txid = $block.tx[$_]
        $rawtx = bitcoin-cli getrawtransaction $txid 1 2>$null | ConvertFrom-Json -ErrorAction SilentlyContinue
        if ($rawtx -and $rawtx.vout) {
            $rawtx.vout | ForEach-Object {
                if ($_.scriptPubKey.asm -match "OP_RETURN") {
                    $data = $_.scriptPubKey.hex -replace "^.*([a-f0-9]{2,})$", '$1'
                    if ($data -match "(clinical|trial|genomic|research|nih|fda)") {
                        Write-Host "  🎯 MEDICAL OP_RETURN: $($_.scriptPubKey.asm)" -ForegroundColor Red
                    }
                }
            }
        }
    }
}

# Reload profile
. $PROFILE
