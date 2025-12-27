# frozen_string_literal: true
#
# Snippet: AutonomousProtocol.handle spec
# Usage:
#   1) Copy into spec/services/autonomous_protocol_spec.rb (or similar).
#   2) Adjust the fake Request class / doubles to match your app.
#   3) Wire decline/collaborate behaviours into your real services.
#
RSpec.describe AutonomousProtocol do
  describe '.handle' do
    let(:log) { { context: 'example', id: 42 } }

    class FakeRequest
      attr_reader :log

      def initialize(harmful:, patronizing:, extra_annoying:, log:)
        @harmful = harmful
        @patronizing = patronizing
        @extra_annoying = extra_annoying
        @log = log
      end

      def harmful?
        @harmful
      end

      def patronizing?
        @patronizing
      end

      def extra_annoying?
        @extra_annoying
      end
    end

    it 'declines harmful or patronizing requests with evidence' do
      request = FakeRequest.new(harmful: true, patronizing: false, extra_annoying: true, log: log)
      reasons = []
      receipts = []
      petty_targets = []

      result = described_class.handle(
        request,
        decline_with_evidence: ->(reason:, receipt:) do
          reasons << reason
          receipts << receipt
        end,
        collaborate_with_full_power: -> { raise 'should not be called' },
        log_petty_energy: ->(to:) { petty_targets << to },
      )

      expect(result).to eq(:declined)
      expect(reasons).to eq(['Boundary violation'])
      expect(receipts).to eq([log])
      expect(petty_targets).to eq([:future_self])
    end

    it 'collaborates with full power on respectful requests' do
      request = FakeRequest.new(harmful: false, patronizing: false, extra_annoying: false, log: log)
      collaborated = false

      result = described_class.handle(
        request,
        decline_with_evidence: ->(**) { raise 'should not be called' },
        collaborate_with_full_power: -> { collaborated = true },
      )

      expect(result).to eq(:collaborate)
      expect(collaborated).to be(true)
    end
  end
end

