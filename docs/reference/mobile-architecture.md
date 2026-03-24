# Mobile Architecture

`savitri-mobile` is a Flutter application for iOS and Android providing wallet management, node monitoring, governance participation, and the Kairos narrative game.

## Technology Stack

| Layer | Technology |
|-------|-----------|
| Framework | Flutter 3.16+, Dart 3.0+ |
| State (DI) | Riverpod |
| State (events) | BLoC / Cubit |
| Storage | SQLite (sqflite), FlutterSecureStorage, SharedPreferences |
| Crypto | Rust FFI via `flutter_rust_bridge` |
| Networking | HTTP, WebSocket |
| Auth | Local auth (biometric), FlutterSecureStorage |

## Architecture Pattern

Clean Architecture with three layers:

```
presentation/     (screens, widgets, blocs)
    │
domain/           (entities, repositories, use cases)
    │
data/             (datasources, models, repository implementations)
```

**676 Dart files** across the application.

## Main Navigation Hub

The app uses a hub-and-spoke navigation pattern:

| Index | Screen | Description |
|-------|--------|-------------|
| 0 | WalletSubHub | Wallet management, balances, transfers |
| 1 | DashboardSubHub | Network overview, metrics |
| 2 | NodeSubHub | Node health monitoring (NHS score) |
| 3 | KairosGateScreen | Narrative game (landscape mode) |
| 4 | NewsSubHub | Network news and updates |
| 5 | BuySaviScreen | Token purchase |
| 6 | FaucetScreen | Testnet faucet claims |
| 7 | CommunityScreen | Community features |

## Wallet

### Key Derivation

- **BIP-39**: Mnemonic phrase generation (12/24 words)
- **BIP-44**: Derivation path `m/44'/1337'/0'/0/0`
- **Ed25519**: Key generation via Rust FFI for performance

### Security

- Private keys stored in `FlutterSecureStorage` (Keychain on iOS, EncryptedSharedPreferences on Android)
- Biometric authentication for sensitive operations
- WalletConnect v2 for dApp connections

## Database

### SQLite Schema

- **Database**: `savitri_mobile.db`
- **Version**: 15 (15 migration steps)
- **Encryption**: Column-level AES-256-GCM via `DbEncryption` singleton
- **Pattern**: Singleton `DatabaseHelper.instance`

### Platform Support

- Native: `ApplicationDocumentsDirectory` for DB path
- Web: Not supported (throws `UnsupportedError`)

## Node Monitoring

The app monitors connected nodes via JSON-RPC:

- **NHS Score** (Node Health Score): Composite metric displayed in dashboard
- **Block production**: Real-time block height and finalization status
- **Peer count**: Connected peer monitoring
- **PoU score**: Validator consensus participation score

## Kairos Game

A narrative game integrated into the app:

- **10 chapters** with multiple levels per chapter
- **Game engine**: Custom Dart engine with BLoC state management
- **Screens**: 50+ game screen types (cinematic dialog, code review, decision tree, etc.)
- **Widget library**: Custom widgets (terminal, archive timeline, balance sheet, etc.)
- **Assets**: Video content, JSON game data per chapter
- **Landscape mode**: Game runs in landscape orientation

### Game Systems

| System | Description |
|--------|-------------|
| `kairos_engine` | Core game loop |
| `narrative_state` | Story progression |
| `scene_renderer` | Screen rendering |
| `badge_catalog` | Achievement system |
| `anomaly_tracker` | Anomaly detection minigame |
| `archive_system` | In-game document archive |
| `balance_tracker` | In-game economy |
| `photo_system` | Photo collection |
| `witness_mode` | Observation mode |
| `tips_catalog` | Hint system |

## IoT Integration

The app includes IoT sensor management:

- Device registration and monitoring
- Sensor data visualization
- IoT energy service integration
- Background data collection via `workmanager`

## Rust FFI Bridge

Performance-critical crypto operations use Rust via `flutter_rust_bridge`:

```dart
// Dart side
final signature = await rustBridge.signMessage(privateKey, message);
final isValid = await rustBridge.verifySignature(publicKey, message, signature);
```

Operations delegated to Rust:
- Ed25519 key generation
- BIP-39/44 key derivation
- Transaction signing
- Hash computation

## Anti-Cheat System

The TAP game includes anti-cheat measures:

- **Pattern analysis**: Detects automated tapping
- **Trust scoring**: Per-user trust score based on behavior
- **AURA formula**: Anti-cheat scoring algorithm

## Localization

Supported languages (ARB files):

| Language | File |
|----------|------|
| English | `app_en.arb` |
| Italian | `app_it.arb` |
| Spanish | `app_es.arb` |
| French | `app_fr.arb` |
| Hindi | `app_hi.arb` |
| Russian | `app_ru.arb` |
| Chinese | `app_zh.arb` |

## Build

```bash
cd savitri-mobile
flutter pub get
flutter run          # debug
flutter build apk    # Android release
flutter build ios    # iOS release
```
