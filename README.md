# Rotating-Logger
This Node.js program demonstrates an EventEmitter-based logger with console and file transports and size-based rotation (~50KB per file).
It allows you to log messages, automatically write them to the console and files, and rotate files when they grow too large.

1️⃣ Prerequisites
Node.js v14+

ES modules enabled ("type": "module" in package.json)
{
  "type": "module",
  "scripts": {
    "start": "node logger.mjs"
  }
}
2️⃣ Project Structure
project/
│
├─ logger.mjs          # Main logger code
├─ logs/               # Log files will be created here
└─ package.json
3️⃣ Code Explained
import fs from 'fs';
import path from 'path';
import { EventEmitter } from 'events';
Import Node.js modules:
fs → file operations
path → handle file paths
EventEmitter → base for our logger events
Logger Class
class Logger extends EventEmitter {
  log(level, message) {
    const entry = `[${new Date().toISOString()}] [${level.toUpperCase()}] ${message}`;
    this.emit('log', entry);
  }
  info(msg) { this.log('info', msg); }
  warn(msg) { this.log('warn', msg); }
  error(msg) { this.log('error', msg); }
}
Logger extends EventEmitter to emit log events.
log(level, message):
Formats the message with timestamp and level
Emits a log event for all transports to listen
info(), warn(), error() → helper methods for common log levels.
Console Transport
class ConsoleTransport {
  constructor(logger) {
    logger.on('log', entry => console.log(entry));
  }
}
Subscribes to log events from the logger.
Prints each log entry to the console.
File Transport with Rotation
class FileTransport {
  constructor(logger, { folder = 'logs', filename = 'app.log', maxSize = 50 * 1024 } = {}) {
    this.folder = folder;
    this.filename = filename;
    this.maxSize = maxSize;
    this.filePath = path.join(folder, filename);

    if (!fs.existsSync(folder)) fs.mkdirSync(folder, { recursive: true });

    logger.on('log', entry => this.write(entry));
  }
Constructor accepts logger, folder, filename, and max file size (~50KB default).
Creates logs folder if it doesn’t exist.
Subscribes to logger’s log events to write them to file
  write(entry) {
    const entrySize = Buffer.byteLength(entry + '\n');
    let currentSize = 0;

    if (fs.existsSync(this.filePath)) {
      currentSize = fs.statSync(this.filePath).size;
    }

    if (currentSize + entrySize > this.maxSize) {
      const newFile = path.join(this.folder, `${this.filename}.${Date.now()}`);
      fs.renameSync(this.filePath, newFile);
    }

    fs.appendFileSync(this.filePath, entry + '\n');
  }
}
write(entry):
Calculates size of the new entry.
Checks the current file size.
If adding the entry exceeds maxSize, rotate the file:
Rename current file with a timestamp.
New file will be created automatically.
Append the log entry to the file.
const logger = new Logger();
new ConsoleTransport(logger);
new FileTransport(logger, { folder: 'logs', filename: 'app.log', maxSize: 50 * 1024 });

logger.info('Server started');
logger.warn('Disk almost full');
logger.error('Unhandled exception');
Create a Logger instance.

Attach console and file transports.

Log messages with different levels.

File rotates automatically when it exceeds ~50KB.

4️⃣ Example Output
[2025-10-04T12:00:00.123Z] [INFO] Server started
[2025-10-04T12:00:01.456Z] [WARN] Disk almost full
[2025-10-04T12:00:02.789Z] [ERROR] Unhandled exception
File:
logs/app.log (rotates when >50KB)

Old file renamed like app.log.1696414800000
