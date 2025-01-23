---
title: Retrieves All Users(TESTING SYNC)
excerpt: >


  | Type | Reason | Card Number (PAN) | Card Object Status |

  | ---- | ------ | ----------------- | ------------------ |

  | Digital | Fraud, Lost, Stolen | New | `status=Verified` |

  | Digital | Renewal | Same, with new expiration date and CVV2 |
  `status=Verified` |

  | Digital + physical | Physical damage | Same | `status=Verified` unless
  program requires verification for reissues then
  `status=DigitalActivePhysicalPending` |

  | Digital + physical | Fraud, Lost, Stolen | New |
  `status=DigitalActivePhysicalPending` |

  | Digital + physical | Renewal | Same, with new expiration |
  `status=ReissuePendingVerification` |

  | Physical | Physical Damage | Same | `status=Verified` unless program
  requires verification for reissues then `status=PendingVerification` |

  | Physical | Fraud, Lost, Stolen | New | `status=PendingVerification` |

  | Physical | Renewal | Same, with new expiration date and CVV2 |
  `status=ReissuePendingVerification` |
api:
  file: my-test.json
  operationId: testingSlugChange
hidden: false
---