TODO:

```# fujinet-summercart64-android
FujiNet + Summercart64 + Android - N64 ROM manager and flashcart interface.

// In SC64StorageDevice::handle_rom_load()
void SC64StorageDevice::handle_rom_load(const SC64CommandPacket &cmd)
{
    // cmd.data contains ROM filename
    std::string rom_path(cmd.data.begin(), cmd.data.end());
    
    // Open file from Android filesystem
    FILE *f = _fs->fopen(rom_path.c_str(), "rb");
    if (!f) {
        resp.status = 1;  // Error
        return;
    }
    
    // Read file into buffer
    fseek(f, 0, SEEK_END);
    _rom_size = ftell(f);
    rewind(f);
    
    _rom_buffer = (uint8_t*)malloc(_rom_size);
    fread(_rom_buffer, 1, _rom_size, f);
    fclose(f);
    
    // Send chunks back to SC64 via USB
    for (uint32_t offset = 0; offset < _rom_size; offset += CHUNK_SIZE) {
        SC64ResponsePacket resp;
        resp.status = 0;
        uint32_t chunk = std::min(CHUNK_SIZE, _rom_size - offset);
        resp.data.insert(resp.data.end(), 
                        _rom_buffer + offset, 
                        _rom_buffer + offset + chunk);
        
        // Send to SC64```
        SYSTEM_BUS.send_response(resp);
    }
}
