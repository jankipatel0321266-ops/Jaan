import 'package:flutter/material.dart';

void main() {
  runApp(const B2BHighValuePayApp());
}

class B2BHighValuePayApp extends StatelessWidget {
  const B2BHighValuePayApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      title: 'B2B Crore Pay',
      theme: ThemeData.dark().copyWith(
        scaffoldBackgroundColor: const Color(0xFF0B0F19),
        primaryColor: const Color(0xFF6366F1),
      ),
      home: const MerchantPayScreen(),
    );
  }
}

class MerchantPayScreen extends StatefulWidget {
  const MerchantPayScreen({super.key});

  @override
  State<MerchantPayScreen> createState() => _MerchantPayScreenState();
}

class _MerchantPayScreenState extends State<MerchantPayScreen> {
  final TextEditingController _businessController = TextEditingController();
  final TextEditingController _gstController = TextEditingController();
  final TextEditingController _amountController = TextEditingController();

  double _corporateBalance = 50000000.00; // ₹5 करोड़ का टेस्ट कॉर्पोरेट बैलेंस

  final List<Map<String, dynamic>> _pendingRequests = [];
  final List<Map<String, dynamic>> _completedTxns = [
    {
      'id': 'RTGS998231',
      'party': 'रिलायंस पॉलीमर इंडस्ट्रीज',
      'gst': 'GSTIN: 24AAACR1234F1Z0',
      'amount': '₹1,80,00,000',
      'mode': 'RTGS Direct Banking',
      'time': 'आज, 11:30 AM'
    }
  ];

  void _sendCrorePaymentRequest() {
    String partyName = _businessController.text.trim();
    String gst = _gstController.text.trim();
    String amountStr = _amountController.text.trim();

    if (partyName.isEmpty || amountStr.isEmpty) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('कृपया व्यापारी का नाम और राशि दर्ज करें।')),
      );
      return;
    }

    double amount = double.tryParse(amountStr) ?? 0.0;

    // Limit check up to 2 Crores (20,00,00,000 / 2,00,00,000)
    if (amount <= 0 || amount > 200000000) {
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('राशि ₹1 से ₹2,00,00,000 (2 करोड़) के बीच होनी चाहिए।')),
      );
      return;
    }

    setState(() {
      _pendingRequests.insert(0, {
        'id': 'REQ${DateTime.now().millisecondsSinceEpoch.toString().substring(6)}',
        'party': partyName,
        'gst': gst.isNotEmpty ? 'GSTIN: $gst' : 'GST: Non-Verified',
        'amount': amount,
        'mode': amount >= 500000 ? 'RTGS (High Value)' : 'IMPS',
        'time': 'Pending Approval'
      });
      _businessController.clear();
      _gstController.clear();
      _amountController.clear();
    });

    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(
        backgroundColor: Colors.indigo,
        content: Text('₹${amount.toStringAsFixed(0)} की इंस्टेंट रिक्वेस्ट भेज दी गई है।'),
      ),
    );
  }

  void _approveRequest(int index) {
    var req = _pendingRequests[index];
    double amt = req['amount'];

    setState(() {
      _corporateBalance += amt;
      _completedTxns.insert(0, {
        'id': req['id'],
        'party': req['party'],
        'gst': req['gst'],
        'amount': '₹${amt.toStringAsFixed(0)}',
        'mode': req['mode'],
        'time': 'अभी'
      });
      _pendingRequests.removeAt(index);
    });

    showDialog(
      context: context,
      builder: (ctx) => AlertDialog(
        backgroundColor: const Color(0xFF1E293B),
        title: const Row(
          children: [
            Icon(Icons.check_circle_sharp, color: Colors.green, size: 28),
            SizedBox(width: 8),
            Text('2 करोड़ ट्रांसफर सफल!'),
          ],
        ),
        content: Text(
          '₹$amt RTGS/Connected Banking के जरिए सीधे बैंक खाते में क्रेडिट हो चुके हैं।',
          style: const TextStyle(color: Colors.white70),
        ),
        actions: [
          TextButton(
            onPressed: () => Navigator.pop(ctx),
            child: const Text('ठीक है', style: TextStyle(color: Colors.indigoAccent)),
          )
        ],
      ),
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        backgroundColor: const Color(0xFF1E293B),
        title: const Text('B2B Crore Direct Pay', style: TextStyle(fontWeight: FontWeight.bold)),
      ),
      body: SingleChildScrollView(
        padding: const EdgeInsets.all(16.0),
        child: Column(
          crossAxisAlignment: CrossAxisAlignment.start,
          children: [
            // Bank Balance
            Container(
              width: double.infinity,
              padding: const EdgeInsets.all(20),
              decoration: BoxDecoration(
                gradient: const LinearGradient(
                  colors: [Color(0xFF312E81), Color(0xFF4F46E5)],
                  begin: Alignment.topLeft,
                  end: Alignment.bottomRight,
                ),
                borderRadius: BorderRadius.circular(16),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('कॉर्पोरेट अकाउंट बैलेंस (RTGS Ready)', style: TextStyle(color: Colors.white70)),
                  const SizedBox(height: 6),
                  Text(
                    '₹${_corporateBalance.toStringAsFixed(2)}',
                    style: const TextStyle(color: Colors.white, fontSize: 28, fontWeight: FontWeight.bold),
                  ),
                  const SizedBox(height: 8),
                  const Text('सुरक्षित - Banking API Encrypted (Max Limit: ₹2,00,00,000)', style: TextStyle(color: Colors.greenAccent, fontSize: 12)),
                ],
              ),
            ),

            const SizedBox(height: 20),

            // Form
            Container(
              padding: const EdgeInsets.all(16),
              decoration: BoxDecoration(
                color: const Color(0xFF1E293B),
                borderRadius: BorderRadius.circular(16),
              ),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                  const Text('बड़ी रकम की पेमेंट रिक्वेस्ट (Up to ₹2 Crore)', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold, fontSize: 15)),
                  const SizedBox(height: 12),
                  TextField(
                    controller: _businessController,
                    style: const TextStyle(color: Colors.white),
                    decoration: const InputDecoration(
                      hintText: 'व्यापारी / फर्म का नाम',
                      hintStyle: TextStyle(color: Colors.grey),
                      filled: true,
                      fillColor: Color(0xFF0F172A),
                      border: InputBorder.none,
                    ),
                  ),
                  const SizedBox(height: 10),
                  TextField(
                    controller: _gstController,
                    style: const TextStyle(color: Colors.white),
                    decoration: const InputDecoration(
                      hintText: 'GSTIN नम्बर (जैसे: 24AAAAA0000A1Z5)',
                      hintStyle: TextStyle(color: Colors.grey),
                      filled: true,
                      fillColor: Color(0xFF0F172A),
                      border: InputBorder.none,
                    ),
                  ),
                  const SizedBox(height: 10),
                  TextField(
                    controller: _amountController,
                    keyboardType: TextInputType.number,
                    style: const TextStyle(color: Colors.white),
                    decoration: const InputDecoration(
                      hintText: 'राशि रुपये में (उदा: 20000000)',
                      hintStyle: TextStyle(color: Colors.grey),
                      filled: true,
                      fillColor: Color(0xFF0F172A),
                      border: InputBorder.none,
                    ),
                  ),
                  const SizedBox(height: 14),
                  SizedBox(
                    width: double.infinity,
                    height: 48,
                    child: ElevatedButton(
                      style: ElevatedButton.styleFrom(backgroundColor: const Color(0xFF4F46E5)),
                      onPressed: _sendCrorePaymentRequest,
                      child: const Text('रिक्वेस्ट भेजें (Instant Notification)', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
                    ),
                  )
                ],
              ),
            ),

            const SizedBox(height: 20),

            // Pending Approvals
            if (_pendingRequests.isNotEmpty) ...[
              const Text('प्राप्त रिक्वेस्ट (Approval Portal)', style: TextStyle(color: Colors.orangeAccent, fontWeight: FontWeight.bold)),
              const SizedBox(height: 8),
              ListView.builder(
                shrinkWrap: true,
                physics: const NeverScrollableScrollPhysics(),
                itemCount: _pendingRequests.length,
                itemBuilder: (ctx, i) {
                  var req = _pendingRequests[i];
                  return Card(
                    color: const Color(0xFF1E293B),
                    child: ListTile(
                      title: Text(req['party'], style: const TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
                      subtitle: Text('${req['gst']} • ${req['mode']}'),
                      trailing: ElevatedButton(
                        style: ElevatedButton.styleFrom(backgroundColor: Colors.green),
                        onPressed: () => _approveRequest(i),
                        child: Text('Approve (₹${req['amount']})'),
                      ),
                    ),
                  );
                },
              ),
            ],

            const SizedBox(height: 20),

            // History
            const Text('ट्रांसफर इतिहास (Settlement History)', style: TextStyle(color: Colors.white, fontWeight: FontWeight.bold)),
            const SizedBox(height: 8),
            ListView.builder(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              itemCount: _completedTxns.length,
              itemBuilder: (ctx, i) {
                var txn = _completedTxns[i];
                return ListTile(
                  tileColor: const Color(0xFF1E293B),
                  title: Text(txn['party'], style: const TextStyle(color: Colors.white)),
                  subtitle: Text('${txn['gst']} • ${txn['time']}'),
                  trailing: Text(txn['amount'], style: const TextStyle(color: Colors.greenAccent, fontWeight: FontWeight.bold, fontSize: 16)),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
